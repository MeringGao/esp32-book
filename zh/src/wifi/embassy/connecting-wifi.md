# 使用 Embassy 支持连接 ESP32 到 Wi-Fi

因此，我们已经讨论了本练习所需的依赖项以及需要启用的功能。接下来，我们将专注于编码。首先，我们将查看使用 Embassy 连接 Wi-Fi 的代码。

## StaticCell 辅助宏

在嵌入式环境中，当您需要在运行时初始化变量但要求其具有静态生命周期（static lifetime）时，StaticCell crate 非常有用。我们将定义一个宏来创建全局可访问的静态变量。此宏接受两个参数：变量的类型和用于初始化它的值。uninit 函数提供对未初始化内存的可变引用，我们将值写入其中。

```rust
// 如果您愿意使用 nightly 编译器，可以使用 static_cell crate 提供的宏：https://docs.rs/static_cell/2.1.0/static_cell/macro.make_static.html

macro_rules! mk_static {
    ($t:ty,$val:expr) => {{
        static STATIC_CELL: static_cell::StaticCell<$t> = static_cell::StaticCell::new();
        #[deny(unused_attributes)]
        let x = STATIC_CELL.uninit().write(($val));
        x
    }};
}
```

## 初始化 Wi-Fi 控制器

让我们用通常的设置初始化 Embassy：

```rust
let timg0 = TimerGroup::new(peripherals.TIMG0);
esp_rtos::start(timg0.timer0);
```

从环境变量加载 Wi-Fi 凭据：
```rust
const SSID: &str = env!("SSID");
const PASSWORD: &str = env!("PASSWORD");
```

接下来，我们用静态生命周期（static lifetime）初始化射频控制器（Radio controller）。我们使用 `mk_static!` 宏将控制器提升为 `'static` 生命周期，因为 Wi-Fi 网络协议栈将作为异步任务运行，在整个程序执行期间持续处理网络事件。这要求 `radio_init` 变量在程序的整个持续时间内保持有效。

```rust
// let radio_init = esp_radio::init().expect("Failed to initialize Wi-Fi/BLE controller");
let radio_init = &*mk_static!(
    esp_radio::Controller<'static>,
    esp_radio::init().expect("Failed to initialize Wi-Fi/BLE controller")
);
```

接下来，我们创建 Wi-Fi 控制器及其关联的网络接口，然后提取 STA 接口。

```rust
let (wifi_controller, interfaces) =
        esp_radio::wifi::new(&radio_init, peripherals.WIFI, Default::default())
            .expect("Failed to initialize Wi-Fi controller");
let wifi_interface = interfaces.sta;
```

### 初始化网络协议栈

我们需要一个随机数用于 TLS 配置和网络协议栈初始化，两者都需要 u64。然而，由于 rng 只生成 u32 值，我们生成两个随机数，并使用位运算将一个放在最高有效位（MSB），另一个放在最低有效位（LSB）：

```rust
let rng = Rng::new();
let net_seed = rng.random() as u64 | ((rng.random() as u64) << 32);
let tls_seed = rng.random() as u64 | ((rng.random() as u64) << 32);
```

让我们使用从 Wi-Fi 控制器获得的网络接口、作为种子的随机数、DHCP 配置以及大小为 3 的协议栈资源（stack resources），从 embassy_net crate 初始化网络协议栈。

```rust
 let dhcp_config = DhcpConfig::default();
// dhcp_config.hostname = Some(String::from_str("implRust").unwrap());

let config = embassy_net::Config::dhcpv4(dhcp_config);
// 初始化网络协议栈
// Init network stack
let (stack, runner) = embassy_net::new(
    wifi_interface,
    config,
    mk_static!(StackResources<3>, StackResources::<3>::new()),
    net_seed,
);
```

接下来，我们将启动两个后台任务：connection_task 将维护 Wi-Fi 连接，而 net_task 将运行网络协议栈并处理网络事件。

```rust
spawner.spawn(connection(controller)).ok();
spawner.spawn(net_task(runner)).ok();
```

我们稍后会讨论这两个任务中发生了什么并检查这些函数定义。但首先，让我们完成流程。

## 等待 Wi-Fi 连接

我们将等待 Wi-Fi 链路启动，然后获取 IP 地址。

```rust
async fn wait_for_connection(stack: Stack<'_>) {
    println!("Waiting for link to be up");
    loop {
        if stack.is_link_up() {
            break;
        }
        Timer::after(Duration::from_millis(500)).await;
    }

    println!("Waiting to get IP address...");
    loop {
        if let Some(config) = stack.config_v4() {
            println!("Got IP: {}", config.address);
            break;
        }
        Timer::after(Duration::from_millis(500)).await;
    }
}
```


## Wi-Fi 连接任务

connection_task 函数通过持续检查状态、配置 Wi-Fi 控制器并在连接丢失或未启动时尝试重新连接来管理 Wi-Fi 连接。

1. 首先，我们检查 Wi-Fi 状态。如果处于 StaConnected 状态，我们等待直到断开连接。如果断开连接，我们进入循环的其他步骤。
2. 我们检查 Wi-Fi 控制器是否已启动。如果没有，我们使用 SSID（Wi-Fi 名称）和密码初始化 Wi-Fi 客户端配置，并启动它。
3. 最后，我们尝试连接到 Wi-Fi。

```rust
#[embassy_executor::task]
async fn connection(mut controller: WifiController<'static>) {
    println!("start connection task");
    println!("Device capabilities: {:?}", controller.capabilities());
    loop {
        match esp_radio::wifi::sta_state() {
            WifiStaState::Connected => {
                // 等待直到不再连接
                // wait until we're no longer connected
                controller.wait_for_event(WifiEvent::StaDisconnected).await;
                Timer::after(Duration::from_millis(5000)).await
            }
            _ => {}
        }
        if !matches!(controller.is_started(), Ok(true)) {
            let client_config = ModeConfig::Client(
                ClientConfig::default()
                    .with_ssid(SSID.into())
                    .with_password(PASSWORD.into()),
            );
            controller.set_config(&client_config).unwrap();
            println!("Starting wifi");
            controller.start_async().await.unwrap();
            println!("Wifi started!");

            println!("Scan");
            let scan_config = ScanConfig::default().with_max(10);
            let result = controller
                .scan_with_config_async(scan_config)
                .await
                .unwrap();
            for ap in result {
                println!("{:?}", ap);
            }
        }
        println!("About to connect...");

        match controller.connect_async().await {
            Ok(_) => println!("Wifi connected!"),
            Err(e) => {
                println!("Failed to connect to wifi: {:?}", e);
                Timer::after(Duration::from_millis(5000)).await
            }
        }
    }
}
```

### 运行网络协议栈

```rust
#[embassy_executor::task]
async fn net_task(mut runner: Runner<'static, WifiDevice<'static>>) {
    runner.run().await
}
```

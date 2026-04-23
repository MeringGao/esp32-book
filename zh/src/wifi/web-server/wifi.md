# Wi-Fi 模块


## Wi-Fi 连接设置

Wi-Fi 设置代码与 ["使用 Embassy 连接 Wi-Fi"](../embassy/connecting-wifi.md) 章节中解释的访问网站相同。为了避免重复，这里不再解释。如果您还没有看过，请参阅该章节。

```rust
const SSID: &str = env!("SSID");
const PASSWORD: &str = env!("PASSWORD");
```

## 启动 Wi-Fi

`start_wifi` 函数负责为 ESP32 设置和启动 Wi-Fi 连接。以下是逐步解释：

1. 我们将创建 STA（Station）模式下的 Wi-Fi 接口和控制器，以便 ESP32 可以连接到现有的 Wi-Fi 网络。

2. 通常，当设备连接到 Wi-Fi 网络时，路由器会自动使用 DHCP 服务器为其分配 IP 地址。在本练习中，我们将配置 ESP32 请求 DHCP 获取 IP。
   为此，我们将创建一个配置为 DHCP 的 `net_config` 实例。然后，我们将使用此配置以及 Wi-Fi 接口来初始化网络协议栈（network stack）和 runner 实例。

3. 我们将生成两个任务：
   - **连接任务（connection task）** 以监控 Wi-Fi 连接并在断开连接时重新连接。
   - **网络任务（network task）** 以管理所有网络通信。

4. 我们将等待 Wi-Fi 链路启动。一旦连接准备就绪，我们将打印分配给我们 ESP32 的 IP 地址。

5. 最后，我们将返回网络协议栈实例。这将稍后被 Web 任务用于处理 Web 服务器操作。

```rust

pub async fn start_wifi(
    radio_init: &'static esp_radio::Controller<'static>,
    wifi: esp_hal::peripherals::WIFI<'static>,
    rng: Rng,
    spawner: &Spawner,
) -> Stack<'static> {
    let (wifi_controller, interfaces) = esp_radio::wifi::new(radio_init, wifi, Default::default())
        .expect("Failed to initialize Wi-Fi controller");

    let wifi_interface = interfaces.sta;
    let net_seed = rng.random() as u64 | ((rng.random() as u64) << 32);

    let dhcp_config = DhcpConfig::default();
    let net_config = embassy_net::Config::dhcpv4(dhcp_config);

    // 初始化网络协议栈
    // Init network stack
    let (stack, runner) = embassy_net::new(
        wifi_interface,
        net_config,
        mk_static!(StackResources<3>, StackResources::<3>::new()),
        net_seed,
    );

    spawner.spawn(connection(wifi_controller)).ok();
    spawner.spawn(net_task(runner)).ok();

    wait_for_connection(stack).await;

    stack
}

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


### Wi-Fi 和网络任务

这两个任务的逻辑没有重大变化。唯一的区别是现在我们将 runner 实例传递给 net_task，与之前不同。

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

```rust
#[embassy_executor::task]
async fn net_task(mut runner: Runner<'static, WifiDevice<'static>>) {
    runner.run().await
}
```

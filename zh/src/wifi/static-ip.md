
# 静态 IP 地址

在之前的练习中，我们一直依赖 DHCP 服务器为 ESP32 分配 IP 地址。然而，这并不可靠，因为 IP 地址可能会随时间变化。在许多情况下，我们希望分配一个静态 IP 地址（static IP），以确保 ESP32 始终拥有一个固定、可预测的地址。这使得无需每次重新连接网络时都检查或更新其 IP 地址，从而更容易持续访问设备。

现在，我们将移除与 DHCP 相关的代码，并将 net_config 变量修改为使用静态 IP 地址初始化。

首先，让我们定义从环境变量加载静态 IP 和网关 IP 的常量。IP 地址应采用 CIDR 格式，其中包含 IP 地址和子网掩码（subnet mask）。我们需要指定 IP 地址，后跟斜杠和子网掩码。例如，如果您想为 ESP32 分配 IP 地址 192.168.0.50，则应将其写为 192.168.0.50/24。

> [!Tip]
> 您不能随意分配任何 IP 地址。您需要找到您的 Wi-Fi 路由器正在使用的 IP 范围。为此，您可以在终端中输入 `ip a` 并查找 Wi-Fi 接口旁边的 IP 地址（通常以 `wl` 开头）。例如，如果您的系统 IP 地址是 192.168.0.103，您可以分配从 192.168.0.2 开始的 IP 地址。

## 项目基础

您可以复制我们在上一章中创建的相同项目。

```sh
cp -r webserver-html webserver-base
```


```rust
// IP 地址/子网掩码 例如：STATIC_IP=192.168.0.50/24
const STATIC_IP: &str = env!("STATIC_IP");
const GATEWAY_IP: &str = env!("GATEWAY_IP");
```

您还需要配置网关，尽管本练习不需要它，因为我们不会向互联网发送请求。但是，为将来的练习配置它是一个好习惯。

网关地址通常是您的 Wi-Fi IP 范围中的第一个地址。例如，如果您的 IP 地址范围从 192.168.0.1 到 192.168.0.255，则网关很可能是 192.168.0.1。您也可以在 Linux 中使用命令 `ip route | grep default` 来查找您的网关地址。

在我们在上一章中创建的 wifi.rs 模块中，移除 dhcp 配置并在 start_wifi 函数中用以下代码替换它：

```rust
// 额外导入
// Additional imports
use core::str::FromStr;
use core::net::Ipv4Addr;
use embassy_net::Ipv4Cidr;
```

```rust
// 找到 `let net_config` 部分并替换
//find the `let net_config` part and replace
let Ok(ip_addr) = Ipv4Cidr::from_str(STATIC_IP) else {
    println!("Invalid STATIC_IP");
    loop {}
};

let Ok(gateway) = Ipv4Addr::from_str(GATEWAY_IP) else {
    println!("Invalid GATEWAY_IP");
    loop {}
};

let net_config = embassy_net::Config::ipv4_static(StaticConfigV4 {
    address: ip_addr,
    gateway: Some(gateway),
    dns_servers: Vec::new(),
});
// 您不需要更改 `embassy_net::new` 调用中的任何内容。
// You dont need to change anything in `embassy_net::new` call.
```

## 更新后的 start_wifi 函数

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

    let Ok(ip_addr) = Ipv4Cidr::from_str(STATIC_IP) else {
        println!("Invalid STATIC_IP");
        loop {}
    };

    let Ok(gateway) = Ipv4Addr::from_str(GATEWAY_IP) else {
        println!("Invalid GATEWAY_IP");
        loop {}
    };

    let net_config = embassy_net::Config::ipv4_static(StaticConfigV4 {
        address: ip_addr,
        gateway: Some(gateway),
        dns_servers: Default::default(),
    });
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
```

## 项目基础

我已经通过将项目拆分为 wifi 和 web 等模块来重新组织项目，以保持主文件整洁。我们将使用这个项目作为即将到来的练习的基础，因此我建议您看一下它是如何组织的。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/webserver-base
```

**项目结构**：

```
├── build.rs
├── Cargo.toml
├── rust-toolchain.toml
├── src
│   ├── bin
│   │   └── main.rs
│   ├── index.html
│   ├── lib.rs
│   ├── web.rs
│   └── wifi.rs
```

### 如何运行？

通常，我们只需运行 `cargo run --release`，但这次我们还需要传递 Wi-Fi 连接的环境变量。

```sh
SSID=YOUR_WIFI_NAME PASSWORD=YOUR_WIFI_PASSWORD STATIC_IP=ASSIGN_ESP32_IP/24 GATEWAY_IP=WIFI_GATEWAY_IP  cargo run --release
```

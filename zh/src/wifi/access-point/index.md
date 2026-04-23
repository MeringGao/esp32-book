# 接入点 - 在 ESP32 上创建 Wi-Fi 网络

到目前为止，我们一直在使用现有的 Wi-Fi 网络。但是，您可以使用 ESP32 创建自己的 Wi-Fi 网络（只是不要期望它提供互联网）。在本练习中，我们将把 ESP32 配置为接入点（Access Point）并运行 Web 服务器。


## 项目基础

我们将再次复制 webserver-base 项目并在此基础上进行开发。

我建议您在继续之前阅读以下部分；这将避免不必要的代码和解释重复。
- [创建 Web 服务器](../web-server/index.md)


```sh
git clone https://github.com/ImplFerris/esp32-projects
cp -r esp32-projects/webserver-base ~/YOUR_PROJECT_FOLDER/wifi-led
```

## 设置 ESP32 Wi-Fi（在 wifi.rs 文件中）

要使用 ESP32 创建我们自己的 Wi-Fi 网络，我们需要使用 CIDR 表示法设置静态 IP 地址（例如：192.168.2.1/24）并指定网关 IP（例如：192.168.2.1）。我们还需要给出一个 Wi-Fi 名称（SSID），只要它在 32 个字符以内，可以是任何您喜欢的名称。虽然网络的密码是可选的，但我们将使用密码进行设置。SSID 和密码将从环境变量中加载，就像我们在站点（Station）模式下所做的那样。

```rust
// 与站点模式不同，您可以指定任何您喜欢的私有 IP 范围
// IP 地址/子网掩码 例如：STATIC_IP=192.168.13.37/24
const STATIC_IP: &str = "192.168.13.37/24";
// 网关 IP 例如：GATEWAY_IP="192.168.13.37"
const GATEWAY_IP: &str = "192.168.13.37";

const PASSWORD: &str = env!("PASSWORD");
const SSID: &str = env!("SSID");
```

## 更新连接任务

此函数的主要目标是确保 Wi-Fi 网络正在运行。如果它没有运行，它将被重新启动。该函数在循环中检查 Wi-Fi 状态。如果状态是 `WifiApState::Started`，它会等待直到 Wi-Fi 停止（即，发生 `ApStop` 事件）。如果发生这种情况，它会进入循环的第二部分。

如果 Wi-Fi 未启动，我们将使用 SSID、可选密码和 WPA2 Personal 认证模式配置 Wi-Fi。然后，我们将启动 Wi-Fi 网络。

**注意：**

如果您想在没有密码的情况下运行 Wi-Fi，您可以在 `AccessPointConfig` 中注释掉 `password` 和 `auth_method` 构建器。这将使 Wi-Fi 网络无密码，将使用默认配置。


```rust
#[embassy_executor::task]
async fn connection(mut controller: WifiController<'static>) {
    println!("start connection task");
    println!("Device capabilities: {:?}", controller.capabilities());
    loop {
        match esp_radio::wifi::ap_state() {
            WifiApState::Started => {
                // 等待直到 Wi-Fi 停止
                // wait until we're no longer connected
                controller.wait_for_event(WifiEvent::ApStop).await;
                Timer::after(Duration::from_millis(5000)).await
            }
            _ => {}
        }

        if !matches!(controller.is_started(), Ok(true)) {
            let client_config = ModeConfig::AccessPoint(
                AccessPointConfig::default()
                    .with_ssid(SSID.into())
                    .with_password(PASSWORD.into())
                    .with_auth_method(esp_radio::wifi::AuthMethod::Wpa2Personal),
            );
            controller.set_config(&client_config).unwrap();
            println!("Starting wifi");
            controller.start_async().await.unwrap();
            println!("Wifi started!");
        }
    }
}
```

## 更新 start_wifi 函数

我们需要更改接口以使用接入点（Access Point）模式而不是站点（Station）模式。

```rust
let wifi_interface = interfaces.ap;
```

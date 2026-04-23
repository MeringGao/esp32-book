
# 如何运行？

我们需要传递 Wi-Fi 连接的环境变量。

注意：这次，您需要为我们使用 ESP32 创建的网络提供 Wi-Fi 名称和密码。您可以选择最多 32 个字符的任何名称并传递密码（或者如果您在代码中注释掉了密码部分，则可以跳过它）。

```sh
SSID=YOUR_WIFI_NAME PASSWORD=YOUR_WIFI_PASSWORD  cargo run --release
```

将程序烧录（flash）到 ESP32 后，等待它打印 IP 地址并显示 Web 服务器已启动。

## 将您的系统连接到 ESP32 的 Wi-Fi

我们没有在 ESP32 上运行任何 DHCP 服务器，因此连接到我们的 Wi-Fi 网络的设备不会自动获取 IP。因此，我们在连接该设备时需要配置静态 IP 地址。您可以查找如何在您的操作系统上为 Wi-Fi 设置静态 IP 地址并进行相应配置。

确保您的系统连接到我们创建的 Wi-Fi 网络。然后，您可以通过在浏览器中导航到 "http://192.168.13.37/"（替换为您分配的 IP 地址）来访问网页。

<img style="display: block; margin: auto;" src="../images/access-point.png"/>


## 将您的手机连接到 ESP32 的 Wi-Fi

当将您的 Android 手机连接到 ESP32 Wi-Fi 时，选择 Wi-Fi 网络，您可以点击 "更多详情"（这可能因手机型号而异），设置静态 IP，然后重新连接。

<img style="display: block; margin: auto;width:400px;" src="../images/mobile-static-ip.jpg"/>

在 Android 上，您可能会看到类似 "互联网可能不可用" 的错误消息。在这种情况下，选择 "仅连接这一次" 或 "始终连接" 选项。连接后，您可以通过导航到 URL 来访问 Web 服务器。

<img style="display: block; margin: auto;width:400px;" src="../images/mobile-allow-wifi.jpg"/>

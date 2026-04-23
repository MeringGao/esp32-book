# Wi-Fi

到目前为止，我们还没有讨论 ESP32 芯片最大的优势之一：Wi-Fi 支持。在本节中，我们将探索 ESP32 的 Wi-Fi 功能以及我们可以用它实现什么。

ESP32 支持标准 Wi-Fi 通信协议（802.11 b/g/n），并可以在两种模式下运行：站点（Station，STA）模式和软接入点（Soft Access Point）模式。它还能够同时运行这两种模式。


## 站点（Station，STA）模式

在这种模式下，ESP32 作为客户端连接到现有的 Wi-Fi 网络，类似于你的智能手机或笔记本电脑连接到 Wi-Fi 路由器。连接后，ESP32 可以访问互联网或与同一网络上的其他设备通信。

<img style="display: block; margin: auto;" alt="ESP32 Wi-Fi Station(STA) Mode" src="./images/esp32-station-sta-mode-wifi.png"/>


## 接入点（Access Point，AP）模式

在这种模式下，ESP32 充当 Wi-Fi 接入点（Access Point），创建自己的网络，其他设备可以连接到该网络。ESP32 本质上充当路由器，允许智能手机、笔记本电脑或其他微控制器（例如：ESP32）等设备直接与其通信。

<img style="display: block; margin: auto;" alt="ESP32 Wi-Fi Access Point (AP) Mode" src="./images/esp32-wifi-access-point-mode.png"/>

# 使用网页控制 ESP32 LED

我们将创建一个简单的 API 端点（endpoint），接受布尔输入来控制 LED 的状态。与此一起，将提供一个 "index.html" 页面，显示两个按钮：一个用于打开 LED，另一个用于关闭 LED。

当您按下其中一个按钮时，将向 "/led" 端点发送一个请求，并附带以下 JSON 载荷（payload）：

- `{ "is_on": true }` 打开 LED
- `{ "is_on": false }` 关闭 LED

根据 "is_on" 字段的值，led 模块的 "LED_STATE" 变量将被更新。然后 "led_task" 将相应地打开或关闭 LED。

## 路由（Routing）

在 "build_app" 函数中，我们为应用程序配置 Web 路由。根路径（"/"）将提供 "index.html" 内容，我们必须将此文件放在 "src/" 文件夹中。"/led" 路径将接受 "POST" 请求，并由 "led_handler" 处理。

```rust
pub struct Application;

impl AppBuilder for Application {
    type PathRouter = impl routing::PathRouter;

    fn build_app(self) -> picoserve::Router<Self::PathRouter> {
        picoserve::Router::new()
            .route(
                "/",
                routing::get_service(File::html(include_str!("index.html"))),
            )
            .route("/led", routing::post(led_handler))
    }
}
```

## LED 处理程序

我们将定义两个结构体，一个用于处理传入输入，另一个用于发送响应。LedRequest 结构体将派生 Deserialize 以解析传入的 JSON 并将其作为结构体实例提供。LedResponse 结构体将派生 Serialize 以转换结构体实例并将其作为 JSON 响应发送。

```rust
#[derive(serde::Deserialize)]
struct LedRequest {
    is_on: bool,
}

#[derive(serde::Serialize)]
struct LedResponse {
    success: bool,
}
```

在 led_handler 函数中，LedRequest 作为参数被提取。我们可以直接将 "is_on" 值存储在 LED_STATE 中，因为两者都是布尔值。最后，处理程序将返回一个带有 LedResponse 的 JSON 响应，指示成功。

```rust
async fn led_handler(input: picoserve::extract::Json<LedRequest>) -> impl IntoResponse {
    crate::led::LED_STATE.store(input.0.is_on, Ordering::Relaxed);

    picoserve::response::Json(LedResponse { success: true })
}
```


## 网页内容

您可以从 [此处](https://github.com/ImplFerris/esp32-projects/blob/main/wifi-led/src/index.html) 下载 index.html 文件并将其放在 "src/" 文件夹中，或者创建您自己的自定义内容来发送 JSON 请求。

**注意：**

您需要将 URL "http://192.168.0.50/led" 更新为您的 ESP32 的 IP 地址。我在这里为了简单起见将其硬编码；否则，我们需要使用占位符并动态替换它，或者采用基于模板的方法。

```html
<div class="button-container">
        <button class="btn-on" onclick="sendRequest(true)">Turn on LED</button>
        <button class="btn-off" onclick="sendRequest(false)">Turn off LED</button>
    </div>

    <script>
        function sendRequest(is_on) {
            const url = 'http://192.168.0.50/led'; // Replace with STATIC IP of ESP32

            fetch(url, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ is_on })
            })
            .then(response => {
                if (response.ok) {
                    return response.json();
                }
                throw new Error('Network response was not ok');
            })
            .then(data => {
                console.log('Success:', data);
                //alert(LED turned ${action});
            })
            .catch(error => {
                console.error('Error:', error);
                alert('Failed to send the request');
            });
        }
    </script>
```



## 克隆现有项目

您也可以克隆（或参考）我创建的项目并导航到 `wifi-led` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/wifi-led
```

### 如何运行？

将 Wi-Fi 名称、密码、静态 IP 和网关 IP 地址作为环境变量传递，然后烧录（flash）ESP32。

```sh
SSID=YOUR_WIFI_NAME PASSWORD=YOUR_WIFI_PASSWORD STATIC_IP=ASSIGN_ESP32_IP/24 GATEWAY_IP=WIFI_GATEWAY_IP  cargo run --release
```

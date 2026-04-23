# Web 模块 - 提供网页服务

我们已经完成了 Wi-Fi 连接的样板代码。接下来，我们将使用 picoserve crate 为根 URL（"/"）设置一个路由，该路由将提供我们的 HTML 页面。

## `impl_trait_in_assoc_type` 功能

picoserve crate 需要使用 `impl_trait_in_assoc_type` 功能，这是 Rust 中目前的一个不稳定功能（unstable feature）。要启用此功能，您需要在 lib.rs 文件顶部添加以下行（我们已经这样做了）：

```rust
#![feature(impl_trait_in_assoc_type)]
```

## 应用程序和路由

picoserve crate 提供了各种 trait 来配置 Web 应用程序所需的路由和其他功能。`AppBuilder` trait 用于创建无状态的静态路由器，而 `AppWithStateBuilder` trait 允许创建带有应用程序状态的静态路由器。由于我们的应用程序只提供一个 HTML 页面且不需要状态，我们将实现 AppBuilder trait。您可以在 [此处](https://github.com/sammhicks/picoserve/tree/development/examples) 找到更多关于如何使用 picoserve 的示例。

```rust
pub struct Application;

impl AppBuilder for Application {
    type PathRouter = impl routing::PathRouter;

    fn build_app(self) -> picoserve::Router<Self::PathRouter> {
        picoserve::Router::new().route(
            "/",
            routing::get_service(File::html(include_str!("index.html"))),
        )
    }
}
```

我们创建了一个实现 AppBuilder trait 的简单结构体。我们需要指定 PathRouter 类型，我们将其定义为任何实现了 routing::PathRouter trait 的类型。

然后，我们需要实现 build_app 函数，该函数返回一个 Router 实例。我们为 "/" 设置了一个单一的路由，用于提供静态 HTML 页面。HTML 页面的内容在编译时使用 include_str!("index.html") 宏嵌入到应用程序中。将 "index.html" 文件放在 "src/" 文件夹中。


## 池大小（Pool size）

我们需要启动多个任务来处理传入请求。很快，我们将创建一个 web_task 函数（一个 Embassy 任务），其池大小由我们现在定义的常量值设置。然后，我们将根据此值在循环中启动任务，该值控制可以同时运行多少个任务。

```rust
pub const WEB_TASK_POOL_SIZE: usize = 2;
```

我们将池大小设置为 2。在 picoserve 的示例中，池大小设置为 8。您可以增加池大小，但请记住，您还需要相应地调整套接字（socket）和内存 arena 等资源。


## Web 应用程序

我们将定义一个 WebApp 结构体，其中包含 picoserve 路由器的一个实例和配置。

```rust
pub struct WebApp {
    pub router: &'static Router<<Application as AppBuilder>::PathRouter>,
    pub config: &'static picoserve::Config<Duration>,
}
```

接下来，我们为 WebApp 实现 Default trait 并通过调用 build_app 初始化 picoserve Router。我们还配置服务器超时以控制读取请求、等待读取或等待写入响应等操作的持续时间。如果任何操作超过超时时间，连接将被关闭。


```rust
impl Default for WebApp {
    fn default() -> Self {
        let router = picoserve::make_static!(AppRouter<Application>, Application.build_app());

        let config = picoserve::make_static!(
            picoserve::Config<Duration>,
            picoserve::Config::new(picoserve::Timeouts {
                start_read_request: Some(Duration::from_secs(5)),
                read_request: Some(Duration::from_secs(1)),
                write: Some(Duration::from_secs(1)),
                persistent_start_read_request: Some(Duration::from_secs(1)),
            })
            .keep_connection_alive()
        );

        Self { router, config }
    }
}
```

## Web 任务函数

我们创建了一个 Embassy 任务，并在属性中指定了池大小。Web 服务器将监听端口 80。对于每个任务，我们定义 TCP 读取和写入缓冲区以及 HTTP 缓冲区。最后，我们调用 picoserve 的 listen_and_serve 函数来处理传入请求。

```rust
#[embassy_executor::task(pool_size = WEB_TASK_POOL_SIZE)]
pub async fn web_task(
    task_id: usize,
    stack: Stack<'static>,
    router: &'static AppRouter<Application>,
    config: &'static picoserve::Config<Duration>,
) -> ! {
    let port = 80;
    let mut tcp_rx_buffer = [0; 1024];
    let mut tcp_tx_buffer = [0; 1024];
    let mut http_buffer = [0; 2048];

    picoserve::Server::new(router, config, &mut http_buffer)
        .listen_and_serve(task_id, stack, port, &mut tcp_rx_buffer, &mut tcp_tx_buffer)
        .await
        .into_never()
}
```


## 克隆现有项目

您也可以克隆（或参考）我创建的项目并导航到 `webserver-html` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/webserver-html
```

### 如何运行？

对于这个示例，我们也需要传递 Wi-Fi 连接的环境变量。您可以创建一个 .env 文件或直接传递它们，就像我在这里做的那样。

```sh
SSID=YOUR_WIFI_NAME PASSWORD=YOUR_WIFI_PASSWORD  cargo run --release
```

将程序烧录（flash）到 ESP32 后，您应该在控制台中看到以下输出，其中包含您的 Wi-Fi 路由器分配的 IP 地址。

<img style="display: block; margin: auto;" src="../images/wifi-webserver-esp32-output.png"/>

确保您的系统连接到同一个 Wi-Fi 网络。然后，您可以通过在浏览器中导航到 "http://192.168.0.101/"（替换为您收到的 IP 地址）来访问网页。

<img style="display: block; margin: auto;" src="../images/website running on ESP32 with Rust code.png"/>

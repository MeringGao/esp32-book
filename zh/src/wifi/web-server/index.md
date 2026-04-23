# 编写 Rust 代码在 ESP32 上运行网站

在之前的练习中，我们访问了网站并在控制台中打印响应。在本练习中，我们将做相反的事情：我们将在 ESP32 上运行一个 Web 服务器。该服务器可以在本地 Wi-Fi 网络上访问。要使其可以从互联网访问，需要额外的设置。首先，我们将专注于在本地 Wi-Fi 网络内访问该站点。我们仍将仅在 STA 模式下工作（即连接到现有 Wi-Fi）。

## 我们将要做什么

我们将设置一个简单的 Web 服务器来提供单个 index.html 页面。对于此示例，假设 ESP32 已被分配 IP 地址 "192.168.0.101"（运行时会显示在控制台中）。服务器运行后，您可以通过在计算机浏览器中导航到 'http://192.168.0.101/' 来访问该页面。您可以使用自己的 HTML 页面或我为此练习创建的 index.html 页面，您可以在 [此处](https://github.com/ImplFerris/esp32-projects/blob/main/webserver-html/src/bin/index.html) 找到。

> [!Note]
> 您可以设置静态 IP 地址，而不是让 DHCP 服务器分配它。这使 IP 地址保持一致，但会增加一些额外的步骤。为了保持简单，我们不在本练习中这样做。我们将在后面的练习中向您展示如何设置静态 IP。

不再等待，让我们立即开始。

## 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 webserver-html
```

这将打开一个屏幕，要求您选择选项。为了启用 Wi-Fi，我们首先需要启用 "unstable" 和 "alloc" 功能。如果您注意到了，在选择这两个选项之前，您将无法启用 Wi-Fi 选项。因此请逐一选择：

- 首先，选择 "Enable unstable HAL features."
- 选择 "Enable allocations via the esp-alloc crate."
- 现在，您可以启用 "Enable Wi-Fi via esp-radio crate."
- 选择 "Adds embassy framework support"。

还要启用日志记录功能：

- 滚动到 "Flashing, logging and debugging (espflash)" 并按 Enter。
- 然后，选择 "Use defmt to print messages"。

只需按键盘上的 "s" 保存即可。


## 更新依赖项

### picoserve crate

[picoserve](https://docs.rs/picoserve/latest/picoserve/) 是一个为裸机（bare-metal）环境提供异步 HTTP 服务器的 crate，深受 Axum 的启发。正如您可能从名字中猜到的那样，它最初是为 "Raspberry Pi Pico W" 和 Embassy 创建的。但它与其他嵌入式运行时和硬件（包括 ESP32）配合良好。这个 crate 让我们的生活变得更加轻松。没有它，我们将不得不从头开始构建 Web 服务器核心，这是一项耗时的任务，超出了本书的范围。

```toml
picoserve = { version = "0.17.1", features = ["embassy"] }
```

## 项目结构

我们将通过将逻辑拆分为模块来组织代码。在 lib 下，我们将创建两个子模块：web.rs 和 wifi.rs。

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

## Lib 模块

我们将把 mk_static 宏（允许创建在运行时初始化的静态变量）重新定位到 lib 模块中。此外，我们将启用 impl_trait_in_assoc_type 功能，因为 picoserve crate 需要它。

```rust
#![no_std]
#![feature(impl_trait_in_assoc_type)]

pub mod web;
pub mod wifi;

#[macro_export]
macro_rules! mk_static {
    ($t:ty,$val:expr) => {{
        static STATIC_CELL: static_cell::StaticCell<$t> = static_cell::StaticCell::new();
        #[deny(unused_attributes)]
        let x = STATIC_CELL.uninit().write(($val));
        x
    }};
}
```

## main 函数（main.rs 文件）

为了使用 lib 模块，我们通常必须使用完整的项目名称来引用它（例如，webserver::web）。但是，为了保持不同练习之间的引用一致，我们将导入别名为 "lib"。这允许我们使用 "lib::web" 而不是完整的项目名称。

在 main 函数中，我们从一些样板代码开始，设置全局堆分配器（global heap allocator）并初始化 Embassy。

接下来，我们创建一个 Wi-Fi 控制器，将其传递给 start_wifi 函数；我们稍将在 wifi 模块中定义该函数。此函数将返回网络协议栈（network stack）实例。

我们将创建一个 Web 应用程序实例，使用 picoserve crate 配置路由和设置。然后，我们将根据定义的池大小（pool size）生成多个任务来处理传入请求。每个任务接收任务 ID、应用程序实例、网络协议栈和服务器设置。

```rust
use webserver_html as lib;

#[esp_rtos::main]
async fn main(spawner: Spawner) -> ! {
    // generator version: 1.0.0

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    esp_alloc::heap_allocator!(#[unsafe(link_section = ".dram2_uninit")] size: 98767);

    let timg0 = TimerGroup::new(peripherals.TIMG0);
    esp_rtos::start(timg0.timer0);

    info!("Embassy initialized!");

    let radio_init = &*lib::mk_static!(
        esp_radio::Controller<'static>,
        esp_radio::init().expect("Failed to initialize Wi-Fi/BLE controller")
    );
    let rng = Rng::new();

    let stack = lib::wifi::start_wifi(radio_init, peripherals.WIFI, rng, &spawner).await;

    let web_app = lib::web::WebApp::default();
    for id in 0..lib::web::WEB_TASK_POOL_SIZE {
        spawner.must_spawn(lib::web::web_task(
            id,
            stack,
            web_app.router,
            web_app.config,
        ));
    }

    loop {
        Timer::after(Duration::from_secs(1)).await;
    }
}
```

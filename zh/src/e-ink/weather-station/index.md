# 使用 Rust 编写代码，利用电子纸（E-Paper）和 ESP32 构建气象站仪表盘（Weather Dashboard）

让我们用电子墨水屏做一些比仅仅显示静态文字或图像更有趣的事情。我们将构建一个简单的气象站（weather station），它使用 HTTP API 从互联网获取实时天气数据，并用最新信息更新显示屏。这就像是电子纸的 "Hello, World" 程序。

## 先决条件

我们将使用 "openweathermap.org" 的 API 来获取天气数据。该网站提供免费的 API 密钥，每天限制 1000 次请求，应该足够了。前往他们的网站，注册一个免费账户，你就会获得自己的 API 密钥。

### 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 weather-station
```

这将打开一个屏幕，要求你选择选项。

- 首先，选择 "Enable unstable HAL features."
- 选择 "Enable allocations via the esp-alloc crate."
- 现在，你可以启用 "Enable Wi-Fi via esp-radio crate."
- 选择 "Adds embassy framework support".

只需按键盘上的 "s" 保存即可。

## 依赖项

更新 embassy-net 并添加 "dns" 特性

```toml
embassy-net = { version = "0.7.1", features = [
  "defmt",
  "dhcpv4",
  "medium-ethernet",
  "tcp",
  "udp",
  "dns",
] }
```

**额外的依赖项：**

让我快速概述一下新的依赖项。我们已经多次使用过 `embedded-graphics` 和 `tinybmp` 了，包括上一章，所以你现在应该对它们相当熟悉。我们将用它们来绘制形状和加载 bmp 文件。

```toml
embedded-graphics = "0.8.1"
tinybmp = "0.6.0"
profont = "0.7.0"
chrono = { version = "0.4.40", default-features = false, features = ["serde"] }
serde = { version = "1.0.219", default-features = false, features = ["derive"] }
serde-json-core = "0.6.0"
serde_repr = "0.1.20"

epd-waveshare = { features = [
  "graphics",
], git = "https://github.com/ImplFerris/epd-waveshare" }

embedded-hal-bus = { version = "0.3" }

heapless = { version = "0.9.2", features = ["serde"] }

reqwless = { version = "0.13.0", default-features = false, features = [
  "embedded-tls",
  "alloc",
] }
```

- [profont](https://docs.rs/profont/latest/profont/)：该 crate 为 embedded-graphics 提供 ProFont 等宽编程字体。我们使用它是因为气象站仪表盘需要稍大一点的字体尺寸。
- [chrono](https://docs.rs/chrono/latest/chrono/)：用于处理日期和时间。我们将用它来反序列化从 API 接收到的当前日期时间。
- [serde](https://docs.rs/serde/latest/serde/)：一个用于序列化和反序列化 Rust 数据结构的框架。
- [serde_json_core](https://docs.rs/serde-json-core/latest/serde_json_core/)：要将 JSON 反序列化为 Rust 结构体，我们将使用这个 crate。它是专门为 no_std 环境设计的——通常你会使用 serde_json 代替。
- [serde_repr](https://docs.rs/serde_repr/latest/serde_repr/)：该 crate 允许你使用数字值来序列化和反序列化枚举，便于将 API 中的数字天气状况代码（如 200 或 201）映射到 Rust 枚举变体。
- [reqwless](https://docs.rs/reqwless/latest/reqwless/index.html)：我们已经使用过这个 crate，它是一个在 no_std 环境中工作的 HTTP 客户端。我们将用它来向 API 发送请求和接收响应。
- [epd-waveshare](https://docs.rs/epd-waveshare/latest/epd_waveshare/)：一个用于通过 SPI 控制 Waveshare 电子墨水屏的简单驱动。原始 crate 与 1.54 英寸型号配合不正常，所以我不得不 fork 它并修补代码。这更像是一个 hack 而不是适当的修复，所以我还没有向原始仓库发送 PR。目前，我们将使用 fork 版本。

## 项目结构

这是整体项目结构，我们将逐步介绍。

```
├── build.rs
├── src
│   ├── bin
│   │   └── main.rs
│   ├── ca_cert.pem
│   ├── dashboard.rs
│   ├── icons
│   │   ├── air.bmp
│   │   ├── ...bmp
│   │   └── ...bmp
│   ├── icons.rs
│   ├── lib.rs
│   ├── weather.rs
│   └── wifi.rs
├── ...

```

由于这将是一个很大的章节，我建议参考这里完成的项目 (https://github.com/ImplFerris/esp32-epaper-weather/) 作为参考。你可能需要查看它以了解我在教程中不会涵盖的导入和其他非重要细节。

## lib 模块

在 `lib.rs` 中，我们定义了子模块以及我们在 Wi-Fi 部分一直使用的辅助宏。这个宏在编译时为 `static` 值保留内存，但允许它在运行时初始化。

```rust
#![no_std]
pub mod dashboard;
pub mod icons;
pub mod weather;
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

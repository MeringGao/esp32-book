# 使用 ESP32 在电子纸（e-Paper / e-ink）显示屏上写入文字

让我们创建一个简单的程序，使用 [epd-waveshare](https://docs.rs/epd-waveshare/latest/epd_waveshare/) crate 在显示模块上绘制文字。然而，在撰写本文时，这个 crate 与 1.54 英寸 V2 显示屏配合使用时未能按预期工作。为了解决这个问题，我不得不应用一些补丁，因此我们将暂时使用我们的分支版本。我还没有提交拉取请求（pull request），因为还需要进一步改进（局部更新未能按预期工作）。尽管如此，我们的分支版本对于我们的练习来说已经足够。一旦问题解决，本章将更新为使用最新的修复版本。

## 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 e-ink-hello
```

这将打开一个屏幕，要求你选择选项。

- 首先，选择 "Enable unstable HAL features."
- 选择 "Adds embassy framework support".

只需按键盘上的 "s" 保存即可。

## 更新 cargo.toml

```toml
epd-waveshare = { features = [
  "graphics",
], git = "https://github.com/ImplFerris/epd-waveshare", branch = "1in54_v2_fix" }
embedded-hal-bus = { version = "0.3" }
embedded-graphics = "0.8.1"
```

到现在，你应该已经熟悉 embedded-hal-bus 和 embedded-graphics crate 了。embedded-hal crate 为微控制器外设提供了标准化接口（如 SPI、I2C），让开发者可以编写可在任何兼容硬件上工作的可复用驱动。基本上，我们将使用它将 esp-hal 提供的 SpiBus 转换为 epd-waveshare 驱动所需的 SpiDevice。

我们将使用 embedded-graphics crate 在显示屏上渲染文字、形状和图像。

## SPI 设置

让我们初始化 SPI 设备以进行 ESP32 和电子纸显示屏之间的通信。这遵循常规设置：首先，我们初始化 SPI 总线，然后使用 embedded-hal-bus 将其转换为 SPI 设备。

```rust
// 初始化 SPI
// Initialize SPI
let spi = Spi::new(
    peripherals.SPI2,
    SpiConfig::default()
        .with_frequency(Rate::from_mhz(4))
        .with_mode(SpiMode::_0),
)
.unwrap()
//CLK
.with_sck(peripherals.GPIO18)
//DIN
.with_mosi(peripherals.GPIO23);
let cs = Output::new(peripherals.GPIO33, Level::Low, OutputConfig::default());
let mut spi_dev = ExclusiveDevice::new(spi, cs, Delay).unwrap();
```

## 初始化电子纸显示屏

现在，让我们通过传递 SPI 设备以及用于控制的相应 GPIO 引脚来初始化电子纸显示屏。

```rust
// 初始化显示屏
// Initialize Display
let busy_in = Input::new(
    peripherals.GPIO22,
    InputConfig::default().with_pull(Pull::None),
);
let dc = Output::new(peripherals.GPIO17, Level::Low, OutputConfig::default());
let reset = Output::new(peripherals.GPIO16, Level::Low, OutputConfig::default());
let mut display = Display1in54::default();
let mut epd = Epd1in54::new(&mut spi_dev, busy_in, dc, reset, &mut Delay, None).unwrap();
```

## 清除显示屏

首先，我们清除电子纸显示屏的内部缓冲区，然后用白色填充显示屏。之后，我们更新屏幕以显示更改。最后，我们添加一个短暂的延时，让显示屏稳定下来。

```rust
epd.clear_frame(&mut spi_dev, &mut Delay).unwrap();
display.clear(Color::White).unwrap();
epd.update_and_display_frame(&mut spi_dev, display.buffer(), &mut Delay)
    .unwrap();
Timer::after(Duration::from_secs(5)).await;
```

## 写入文字

现在，让我们最终在屏幕上的位置 (x=3, y=100) 写入文字 "impl Rust for ESP32"。之后，我们更新并刷新显示屏以显示文字。我们还添加一个短暂的延时以确保更改生效。

```rust
draw_text(&mut display, "impl Rust for ESP32", 3, 100);
epd.update_and_display_frame(&mut spi_dev, display.buffer(), &mut Delay)
    .unwrap();
Timer::after(Duration::from_secs(5)).await;
```

我们使用一个名为 draw_text 的辅助函数，它使用 embedded_graphics crate 的 Text API 以指定字体大小和黑色将文字写入显示缓冲区。

```rust
fn draw_text(display: &mut Display1in54, text: &str, x: i32, y: i32) {
    let text_style = MonoTextStyleBuilder::new()
        .font(&FONT_10X20)
        .text_color(Color::Black)
        .build();

    Text::with_baseline(text, Point::new(x, y), text_style, Baseline::Top)
        .draw(display)
        .unwrap();
}
```

## 休眠时间（Sleep Time）

使用电子墨水屏显示模块时需要遵循的重要预防措施之一是，使用后要么将其置于睡眠模式（sleep mode），要么完全断电。否则，你会损坏显示屏。

```rust
epd.sleep(&mut spi_dev, &mut Delay).unwrap();
```

## 克隆现有项目

你可以克隆（或参考）我创建的项目并导航到 `e-ink-hello` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/e-ink-hello/
```

## 完整代码

```rust
#![no_std]
#![no_main]
#![deny(
    clippy::mem_forget,
    reason = "mem::forget is generally not safe to do with esp_hal types, especially those \
    holding buffers for the duration of a data transfer."
)]

use defmt::info;
use embassy_executor::Spawner;
use embassy_time::{Delay, Duration, Timer};
use embedded_hal_bus::spi::ExclusiveDevice;
use esp_hal::clock::CpuClock;
use esp_hal::gpio::{Input, InputConfig, Level, Output, OutputConfig, Pull};
use esp_hal::timer::timg::TimerGroup;
use esp_println as _;

// SPI
use esp_hal::spi;
use esp_hal::spi::master::Spi;
use esp_hal::time::Rate;

// epd
use epd_waveshare::color::Color;
use epd_waveshare::epd1in54_v2::{Display1in54, Epd1in54};
use epd_waveshare::prelude::WaveshareDisplay;

// embedded graphics
use embedded_graphics::mono_font::MonoTextStyleBuilder;
use embedded_graphics::mono_font::ascii::FONT_10X20;
use embedded_graphics::prelude::*;
use embedded_graphics::text::{Baseline, Text};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

// 这将创建一个 esp-idf 引导加载程序所需的默认应用描述符。
// 更多信息请参阅：<https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
// This creates a default app-descriptor required by the esp-idf bootloader.
// For more information see: <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
esp_bootloader_esp_idf::esp_app_desc!();

#[esp_rtos::main]
async fn main(spawner: Spawner) -> ! {
    // generator version: 1.0.0

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    let timg0 = TimerGroup::new(peripherals.TIMG0);
    esp_rtos::start(timg0.timer0);

    info!("Embassy initialized!");

    let _ = spawner;

    let spi_bus = Spi::new(
        peripherals.SPI2,
        spi::master::Config::default()
            .with_frequency(Rate::from_mhz(4))
            .with_mode(spi::Mode::_0),
    )
    .unwrap()
    //CLK
    .with_sck(peripherals.GPIO18)
    //DIN
    .with_mosi(peripherals.GPIO23);

    let cs = Output::new(peripherals.GPIO33, Level::Low, OutputConfig::default());
    let mut spi_dev = ExclusiveDevice::new(spi_bus, cs, Delay).unwrap();

    // 初始化显示屏
    // Initialize Display
    let busy_in = Input::new(
        peripherals.GPIO22,
        InputConfig::default().with_pull(Pull::None),
    );
    let dc = Output::new(peripherals.GPIO17, Level::Low, OutputConfig::default());
    let reset = Output::new(peripherals.GPIO16, Level::Low, OutputConfig::default());
    let mut display = Display1in54::default();
    let mut epd = Epd1in54::new(&mut spi_dev, busy_in, dc, reset, &mut Delay, None).unwrap();

    // 清除任何现有图像
    // Clear any existing image
    epd.clear_frame(&mut spi_dev, &mut Delay).unwrap();
    display.clear(Color::White).unwrap();
    epd.update_and_display_frame(&mut spi_dev, display.buffer(), &mut Delay)
        .unwrap();
    Timer::after(Duration::from_secs(5)).await;

    draw_text(&mut display, "impl Rust for ESP32", 3, 100);
    epd.update_and_display_frame(&mut spi_dev, display.buffer(), &mut Delay)
        .unwrap();
    Timer::after(Duration::from_secs(5)).await;

    epd.sleep(&mut spi_dev, &mut Delay).unwrap();

    loop {
        info!("Hello world!");
        Timer::after(Duration::from_secs(1)).await;
    }
}

fn draw_text(display: &mut Display1in54, text: &str, x: i32, y: i32) {
    let text_style = MonoTextStyleBuilder::new()
        .font(&FONT_10X20)
        .text_color(Color::Black)
        .build();

    Text::with_baseline(text, Point::new(x, y), text_style, Baseline::Top)
        .draw(display)
        .unwrap();
}
```

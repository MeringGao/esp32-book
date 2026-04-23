# 使用 ESP32 在 TFT 显示屏上写入文字

让我们创建一个简单的程序，使用 [ili9341](https://docs.rs/ili9341/0.6.0/ili9341/) crate 在显示模块上绘制文字。

## 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 tft-display-hello
```

这将打开一个屏幕，要求你选择选项。

- 首先，选择 "Enable unstable HAL features."

只需按键盘上的 "s" 保存即可。

## 更新 cargo.toml

```toml
embedded-hal-bus = { version = "0.3" }
display-interface-spi = "0.5"
ili9341 = "0.6.0"
embedded-graphics = "0.8.1"
profont = "0.7.0"
```

到现在，你应该已经熟悉 embedded-hal-bus 和 embedded-graphics crate 了。embedded-hal crate 为微控制器外设提供了标准化接口（如 SPI、I2C），让开发者可以编写可在任何兼容硬件上工作的可复用驱动。基本上，我们将使用它将 esp-hal 提供的 SpiBus 转换为 SpiDevice。

然而，与前几章我们直接使用 SpiDevice 不同，ili9341 crate 需要多一层：它期望一个实现了 display-interface-spi crate 中 trait 的接口。这个 crate 定义了 trait 和包装器，通过内部处理数据/命令（DC）引脚等细节来桥接 SPI 总线驱动和显示驱动。

我们使用 profont crate 来获取更大的等宽字体用于我们的显示屏，因为内置的 embedded-graphics 字体太小。

我们将使用 embedded-graphics crate 在显示屏上渲染文字、形状和图像。

## 必要的导入

```rust

// 常用导入
// Usual imports
use defmt::info;
use esp_hal::clock::CpuClock;
use esp_hal::main;
use esp_hal::time::{Duration, Instant};
use esp_println as _;

// Embedded Graphics 相关
// Embedded Grpahics related
use embedded_graphics::mono_font::MonoTextStyle;
use embedded_graphics::pixelcolor::Rgb565;
use embedded_graphics::prelude::*;
use embedded_graphics::text::{Baseline, Text};

// 更大的字体
// Larger font
use profont::{PROFONT_18_POINT, PROFONT_24_POINT};

// ESP32 SPI + 显示驱动桥接
// ESP32 SPI + Display Driver bridge
use display_interface_spi::SPIInterface;
use embedded_hal_bus::spi::ExclusiveDevice;
use esp_hal::delay::Delay;
use esp_hal::spi::master::Config as SpiConfig;
use esp_hal::spi::master::Spi;
use esp_hal::spi::Mode as SpiMode;
use esp_hal::time::Rate; // 用于指定 SPI 频率
use ili9341::{DisplaySize240x320, Ili9341, Orientation};

// 用于管理 GPIO 状态
// For managing GPIO state
use esp_hal::gpio::{Level, Output, OutputConfig};
```

## SPI 设置

让我们初始化 SPI 设备以进行 ESP32 和显示屏之间的通信。这遵循常规设置：首先，我们初始化 SPI 总线，然后使用 embedded-hal-bus 将其转换为 SPI 设备。

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
let cs = Output::new(peripherals.GPIO15, Level::Low, OutputConfig::default());
let dc = Output::new(peripherals.GPIO2, Level::Low, OutputConfig::default());
let reset = Output::new(peripherals.GPIO4, Level::Low, OutputConfig::default());

let spi_dev = ExclusiveDevice::new_no_delay(spi, cs).unwrap();
let interface = SPIInterface::new(spi_dev, dc);
```

这次，我们增加了一个步骤：我们使用 display-interface-spi crate 创建一个 SPIInterface。这个接口将 SPI 设备和数据/命令（DC）引脚组合成一个单一的抽象。它通过处理如何通过 SPI 发送命令和数据来简化通信。我们将把这个接口传递给 TFT 显示驱动。

## 初始化显示屏

要初始化显示屏，我们将 SPI 接口、复位引脚、延时、方向和屏幕尺寸传递给 Ili9341 驱动。这设置了驱动开始与显示屏工作所需的一切。

```rust
let mut display = Ili9341::new(
        interface,
        reset,
        &mut Delay::new(),
        Orientation::Portrait,
        DisplaySize240x320,
    )
    .unwrap();
```

我们将方向设置为竖屏（Portrait），这意味着显示屏被视为宽 240 像素、高 320 像素。显示屏尺寸设置为 240 x 320 像素以匹配屏幕的分辨率（resolution）。这些设置一起帮助驱动根据显示屏的形状和尺寸正确绘制内容。

## 清除显示屏

让我们通过用白色填充背景来清除显示屏。由于 TFT 是彩色显示屏，我们使用 Rgb565 颜色格式，它表示 16 位颜色值（5 位红色、6 位绿色、5 位蓝色）。这是我们第一次使用 Rgb565；到目前为止，我们只使用过单色显示屏。

```rust
display.clear(Rgb565::WHITE).unwrap();
```

## 写入文字

现在，让我们最终在屏幕上显示文字 "impl Rust for ESP32"。我们将使用不同的字体大小和颜色分别写入两部分。

```rust

let text_style = MonoTextStyle::new(&PROFONT_24_POINT, Rgb565::RED);
Text::with_baseline("impl Rust", Point::new(50, 150), text_style, Baseline::Top)
    .draw(&mut display)
    .unwrap();

let text_style = MonoTextStyle::new(&PROFONT_18_POINT, Rgb565::CSS_DIM_GRAY);

Text::with_baseline("for ESP32", Point::new(60, 180), text_style, Baseline::Top)
    .draw(&mut display)
    .unwrap();

```

我们绘制第一行 "impl Rust"，使用红色字体，位置距离左边缘 50 像素，距离屏幕顶部 150 像素。第二行 "for ESP32" 放在它正下方，距离左侧 60 像素，距离顶部 180 像素。如果你写的是不同的文字，可以随意调整坐标以获得最佳的对齐和间距效果。

## 克隆现有项目

你可以克隆（或参考）我创建的项目并导航到 `tft-display-hello` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/tft-display-hello/
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
use esp_hal::clock::CpuClock;
use esp_hal::main;
use esp_hal::time::{Duration, Instant};
use esp_println as _;

// Embedded Graphics 相关
// Embedded Grpahics related
use embedded_graphics::mono_font::MonoTextStyle;
use embedded_graphics::pixelcolor::Rgb565;
use embedded_graphics::prelude::*;
use embedded_graphics::text::{Baseline, Text};

// 更大的字体
// Larger font
use profont::{PROFONT_18_POINT, PROFONT_24_POINT};

// ESP32 SPI + 显示驱动桥接
// ESP32 SPI + Display Driver bridge
use display_interface_spi::SPIInterface;
use embedded_hal_bus::spi::ExclusiveDevice;
use esp_hal::delay::Delay;
use esp_hal::spi::Mode as SpiMode;
use esp_hal::spi::master::Config as SpiConfig;
use esp_hal::spi::master::Spi;
use esp_hal::time::Rate; // 用于指定 SPI 频率
use ili9341::{DisplaySize240x320, Ili9341, Orientation};

// 用于管理 GPIO 状态
// For managing GPIO state
use esp_hal::gpio::{Level, Output, OutputConfig};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

// 这将创建一个 esp-idf 引导加载程序所需的默认应用描述符。
// 更多信息请参阅：<https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
// This creates a default app-descriptor required by the esp-idf bootloader.
// For more information see: <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
esp_bootloader_esp_idf::esp_app_desc!();

#[main]
fn main() -> ! {
    // generator version: 1.0.0

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

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
    let cs = Output::new(peripherals.GPIO15, Level::Low, OutputConfig::default());
    let dc = Output::new(peripherals.GPIO2, Level::Low, OutputConfig::default());
    let reset = Output::new(peripherals.GPIO4, Level::Low, OutputConfig::default());

    let spi_dev = ExclusiveDevice::new_no_delay(spi, cs).unwrap();
    let interface = SPIInterface::new(spi_dev, dc);

    let mut display = Ili9341::new(
        interface,
        reset,
        &mut Delay::new(),
        Orientation::Portrait,
        DisplaySize240x320,
    )
    .unwrap();

    display.clear(Rgb565::WHITE).unwrap();

    let text_style = MonoTextStyle::new(&PROFONT_24_POINT, Rgb565::RED);
    Text::with_baseline("impl Rust", Point::new(50, 150), text_style, Baseline::Top)
        .draw(&mut display)
        .unwrap();

    let text_style = MonoTextStyle::new(&PROFONT_18_POINT, Rgb565::CSS_DIM_GRAY);

    Text::with_baseline("for ESP32", Point::new(60, 180), text_style, Baseline::Top)
        .draw(&mut display)
        .unwrap();

    loop {
        info!("Hello world!");
        let delay_start = Instant::now();
        while delay_start.elapsed() < Duration::from_millis(500) {}
    }
}
```

# 使用 ESP32 和 Rust 在 TFT 显示屏上绘制图像

让我们在 TFT 显示屏上绘制图像。我们可以使用 embedded-graphics crate 来实现这一点，也可以选择使用 tinybmp crate。

我们已经在 OLED 模块部分探索过 tinybmp crate 了。它允许我们加载 BMP 文件并将其显示在屏幕上。或者，你可以使用图像的原始字节数组来实现相同的结果。

该 crate 要求图像为 BMP 格式。如果你的图像采用其他格式，则需要将其转换为 BMP。

我在网上找到了一张不错的图片，但我不确定原作者是谁，无法给予适当的署名。你可以从[这里](./images/embedded-rust.bmp)下载图像。随意使用不同的图像，但请确保将其转换为 BMP 格式。

<img style="display: block; margin: auto;" alt="embedded rust file" src="./images/embedded-rust.bmp"/>


## 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 tft-display-image
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

tinybmp = "0.6.0"
```

我们已经介绍过其他 crate 的详细信息，以及 SPI 和显示模块代码的基本设置。因此，我们不会再次重复这些细节。相反，让我们直接跳到清除屏幕后显示图像的部分。

## 黑色背景

在这个项目中，我们将用黑色填充背景，而不是使用白色背景。

```rust
display.clear(Rgb565::BLACK).unwrap();
```

## 显示图像

将 embedded-rust.bmp 文件放在项目根文件夹内。代码非常简单：将图像加载为字节并传递给 Bmp 的 from_slice 函数。然后，你可以将其与 Image 一起使用。

```rust
let bmp_data = include_bytes!("../../embedded-rust.bmp");
let bmp = Bmp::from_slice(bmp_data).unwrap();

let image = Image::new(&bmp, Point::new(10, 0));
image.draw(&mut display).unwrap();

```

## 克隆现有项目

你可以克隆（或参考）我创建的项目并导航到 `tft-display-image` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/tft-display-image/
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
use embedded_graphics::image::Image;
use esp_hal::clock::CpuClock;
use esp_hal::main;
use esp_hal::time::{Duration, Instant};
use esp_println as _;

// Embedded Graphics 相关
// Embedded Grpahics related
use embedded_graphics::pixelcolor::Rgb565;
use embedded_graphics::prelude::*;

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
use tinybmp::Bmp;

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

    display.clear(Rgb565::BLACK).unwrap();

    let bmp_data = include_bytes!("../../embedded-rust.bmp");
    let bmp = Bmp::from_slice(bmp_data).unwrap();

    let image = Image::new(&bmp, Point::new(10, 0));
    image.draw(&mut display).unwrap();

    loop {
        info!("Hello world!");
        let delay_start = Instant::now();
        while delay_start.elapsed() < Duration::from_millis(500) {}
    }
}
```

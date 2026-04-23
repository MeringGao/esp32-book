# 使用 ESP32 在电子纸（e-Paper / e-ink）显示屏上绘制图像或形状

让我们通过绘制图像和形状来进一步提升电子纸显示屏的效果。我们可以使用 embedded-graphics crate 来实现这一点，也可以选择使用 tinybmp crate。

我们已经在 OLED 模块部分探索过 tinybmp crate 了。它允许我们加载 BMP 文件并将其显示在屏幕上。或者，你可以使用图像的原始字节数组来实现相同的结果。

该 crate 要求图像为 BMP 格式。如果你的图像采用其他格式，则需要将其转换为 BMP。例如，你可以在 Linux 上使用以下命令将 PNG 图像转换为单色 BMP：

```sh
convert ferris.png -monochrome ferris.bmp
```

我已经创建了 Ferris BMP 文件，你可以在本练习中使用。从[这里](./images/ferris.bmp)下载。

<img style="display: block; margin: auto;" alt="ferris bmp file" src="./images/ferris.bmp"/>


## 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 e-ink-image
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
embedded-hal-bus = { version = "0.1" }
embedded-graphics = "0.8.1"
tinybmp = "0.6.0"
```

我们已经介绍过其他 crate 的详细信息，以及 SPI 和显示模块代码的基本设置。因此，我们不会再次重复这些细节。相反，让我们直接跳到清除屏幕后显示图像和形状的部分。

## 黑色背景

在这个项目中，我们将用黑色填充背景，而不是使用白色背景。我选择黑色是因为我们处理的图像具有深色背景，而主体由白色组成。这样看起来效果会更好。

```rust
// 清除任何现有图像
// Clear any existing image
epd.clear_frame(&mut spi_dev, &mut Delay).unwrap();
display.clear(Color::Black).unwrap();
epd.update_and_display_frame(&mut spi_dev, display.buffer(), &mut Delay)
    .unwrap();
Timer::after(Duration::from_secs(5)).await;
```

## 显示图像

将 ferris.bmp 文件放在 src 文件夹内。代码非常简单：将图像加载为字节并传递给 Bmp 的 from_slice 函数。然后，你可以将其与 Image 一起使用。

```rust
let bmp_data = include_bytes!("../ferris.bmp");
let bmp = Bmp::from_slice(bmp_data).unwrap();
let image = Image::new(&bmp, Point::new(25, 60));
image.draw(&mut display).unwrap();

epd.update_and_display_frame(&mut spi_dev, display.buffer(), &mut Delay)
    .unwrap();
Timer::after(Duration::from_secs(5)).await;
```

## 绘制形状

让我们在显示屏上绘制一个圆形。

```rust
// 显示一个圆形
// Display a circle
Circle::new(Point::new(80, 10), 40)
    .into_styled(PrimitiveStyle::with_stroke(Color::White, 2))
    .draw(&mut display)
    .ok();

epd.update_and_display_frame(&mut spi_dev, display.buffer(), &mut Delay)
    .unwrap();
Timer::after(Duration::from_secs(5)).await;
```

## 克隆现有项目

你可以克隆（或参考）我创建的项目并导航到 `e-ink-image` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/e-ink-image/
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
use embedded_graphics::image::Image;
use embedded_graphics::prelude::*;
use embedded_graphics::primitives::{Circle, PrimitiveStyle};

// Bmp
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
    display.clear(Color::Black).unwrap();
    epd.update_and_display_frame(&mut spi_dev, display.buffer(), &mut Delay)
        .unwrap();
    Timer::after(Duration::from_secs(5)).await;

    // 显示图像
    // Display image
    let bmp_data = include_bytes!("../ferris.bmp");
    let bmp = Bmp::from_slice(bmp_data).unwrap();
    let image = Image::new(&bmp, Point::new(25, 60));
    image.draw(&mut display).unwrap();
    epd.update_and_display_frame(&mut spi_dev, display.buffer(), &mut Delay)
        .unwrap();
    Timer::after(Duration::from_secs(5)).await;

    // 显示一个圆形
    // Display a circle
    Circle::new(Point::new(80, 10), 40)
        .into_styled(PrimitiveStyle::with_stroke(Color::White, 2))
        .draw(&mut display)
        .ok();
    epd.update_and_display_frame(&mut spi_dev, display.buffer(), &mut Delay)
        .unwrap();
    Timer::after(Duration::from_secs(5)).await;

    epd.sleep(&mut spi_dev, &mut Delay).unwrap();

    loop {
        info!("Hello world!");
        Timer::after(Duration::from_secs(60)).await;
    }
}
```

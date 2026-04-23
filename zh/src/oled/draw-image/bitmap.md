# 使用位图（Bitmap）图像文件

你可以通过使用 [tinybmp](https://docs.rs/tinybmp/latest/tinybmp/) crate 直接加载 BMP (.bmp) 文件，而无需使用原始图像数据。tinybmp 是一个专为嵌入式环境设计的轻量级 BMP 解析器。虽然它主要用于将 BMP 图像绘制到 embedded_graphics 的 DrawTarget 上，但也可以用于解析 BMP 文件以应用于其他场景。这非常适合我们的需求。

## BMP 文件

该 crate 要求图像必须为 BMP 格式。如果你的图像是其他格式，则需要将其转换为 BMP。例如，你可以在 Linux 上使用以下命令将 PNG 图像转换为单色 BMP：

```sh
convert ferris.png -monochrome ferris.bmp
```

我已经创建了 Ferris 的 BMP 文件，你可以在本练习中使用。从[这里](../images/ferris.bmp)下载。

<img style="display: block; margin: auto;" alt="ferris bmp file" src="../images/ferris.bmp"/>

## 项目基础

我们将复制 old-image 项目并在此基础上进行修改。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cp -r esp32-projects/old-image ~/YOUR_PROJECT_FOLDER/oled-bmp
```

## 更新 Cargo.toml

我们需要一个额外的 crate "tinybmp" 来加载 bmp 图像。

```toml
tinybmp = "0.6.0"

```

## 使用 BMP 文件

将 "ferris.bmp" 文件放在 src 文件夹内。代码非常简单：将图像加载为字节数据，然后传递给 `Bmp` 的 `from_slice` 函数。接着，你就可以将其与 `Image` 一起使用了。

```rust
// 通常的样板代码放在这里...

// 包含 BMP 文件数据。
let bmp_data = include_bytes!("../ferris.bmp");

// 解析 BMP 文件。
let bmp = Bmp::from_slice(bmp_data).unwrap();

// 常规代码：
let image = Image::new(&bmp, Point::new(32, 0));
image.draw(&mut display).unwrap();
display.flush().await.unwrap();
```

## 克隆已有项目

你也可以克隆（或参考）我创建的项目，并导航到 `oled-bmp` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/oled-bmp
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
use embassy_time::{Duration, Timer};
use esp_hal::clock::CpuClock;
use esp_hal::timer::timg::TimerGroup;
use esp_println as _;

// I2C
use esp_hal::i2c::master::Config as I2cConfig; // 为了方便，使用别名导入
use esp_hal::i2c::master::I2c;
use esp_hal::time::Rate;

// OLED
use ssd1306::{I2CDisplayInterface, Ssd1306Async, prelude::*};

// Embedded Graphics
use embedded_graphics::{image::Image, prelude::Point, prelude::*};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

// 创建 ESP-IDF 引导加载程序所需的应用描述符。
// 更多信息请参阅：<https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
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

    let i2c_bus = I2c::new(
        peripherals.I2C0,
        // I2cConfig 是 esp_hal::i2c::master::I2c::Config 的别名
        I2cConfig::default().with_frequency(Rate::from_khz(400)),
    )
    .unwrap()
    .with_scl(peripherals.GPIO18)
    .with_sda(peripherals.GPIO23)
    .into_async();

    let interface = I2CDisplayInterface::new(i2c_bus);

    // 初始化显示屏
    let mut display = Ssd1306Async::new(interface, DisplaySize128x64, DisplayRotation::Rotate0)
        .into_buffered_graphics_mode();
    display.init().await.unwrap();

    // 包含 BMP 文件数据。
    let bmp_data = include_bytes!("../ferris.bmp");

    // 解析 BMP 文件。
    let bmp = tinybmp::Bmp::from_slice(bmp_data).unwrap();

    // 常规代码：
    let image = Image::new(&bmp, Point::new(32, 0));
    image.draw(&mut display).unwrap();
    display.flush().await.unwrap();

    loop {
        Timer::after(Duration::from_secs(1)).await;
    }
}
```

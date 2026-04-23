## 代码

代码中的主要变化是图像数据和宽度。这将在 OLED 上显示 IEC-60617 风格的电阻符号。

## 项目基础

我们将复制 old-image 项目并在此基础上进行修改。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cp -r esp32-projects/old-image ~/YOUR_PROJECT_FOLDER/oled-rawimg
```

## 图像数据

这次，我们将使用以下数据在 OLED 上绘制电阻符号。

```rust
// 31x7 像素
#[rustfmt::skip]
const IMG_DATA: &[u8] = &[
    // 第 1 行
    0b00000001,0b11111111,0b11111111,0b00000000,
    // 第 2 行
    0b00000001,0b11111111,0b11111111,0b00000000,
    // 第 3 行
    0b00000001,0b10000000,0b00000011,0b00000000,
    // 第 4 行
    0b11111111,0b10000000,0b00000011,0b11111110,
    // 第 5 行
    0b00000001,0b10000000,0b00000011,0b00000000,
    // 第 6 行
    0b00000001,0b11111111,0b11111111,0b00000000,
    // 第 7 行
    0b00000001,0b11111111,0b11111111,0b00000000,
];
```

我们需要将宽度设置为 31。我们将在点 (x=35, y=35) 处绘制图像，选择这些坐标并没有什么特别的原因。我只是想展示一下除零点以外的位置。你可以随意尝试不同的点值并探索其他选项。

```rust
 let raw_image = ImageRaw::<BinaryColor>::new(IMG_DATA, 31);

let image = Image::new(&raw_image, Point::new(35, 35));
```

## 克隆已有项目

你也可以克隆（或参考）我创建的项目，并导航到 `oled-rawimg` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/oled-rawimg
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
use embedded_graphics::{
    image::{Image, ImageRaw},
    pixelcolor::BinaryColor,
    prelude::Point,
    prelude::*,
};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

// 创建 ESP-IDF 引导加载程序所需的应用描述符。
// 更多信息请参阅：<https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
esp_bootloader_esp_idf::esp_app_desc!();

// 31x7 像素
#[rustfmt::skip]
const IMG_DATA: &[u8] = &[
    // 第 1 行
    0b00000001,0b11111111,0b11111111,0b00000000,
    // 第 2 行
    0b00000001,0b11111111,0b11111111,0b00000000,
    // 第 3 行
    0b00000001,0b10000000,0b00000011,0b00000000,
    // 第 4 行
    0b11111111,0b10000000,0b00000011,0b11111110,
    // 第 5 行
    0b00000001,0b10000000,0b00000011,0b00000000,
    // 第 6 行
    0b00000001,0b11111111,0b11111111,0b00000000,
    // 第 7 行
    0b00000001,0b11111111,0b11111111,0b00000000,
];

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

    let raw_image = ImageRaw::<BinaryColor>::new(IMG_DATA, 31);
    let image = Image::new(&raw_image, Point::new(35, 35));

    image.draw(&mut display).unwrap();
    display.flush().await.unwrap();

    loop {
        Timer::after(Duration::from_secs(1)).await;
    }
}
```

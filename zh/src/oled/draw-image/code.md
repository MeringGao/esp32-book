## 代码

希望你现在理解了图像在字节数组中的表示方式。接下来，让我们进入编码部分。

## 使用 esp-generate 生成项目

我们将为这个项目启用异步（Embassy）支持。要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 oled-image
```

这将打开一个选项选择界面。

- 选择 "Enable unstable HAL features" 选项
- 然后选择 "Adds embassy framework support" 选项

按键盘上的 "s" 保存即可。

## 更新 Cargo.toml

```toml
ssd1306 = { version = "0.10.0", features = ["async"] }
embedded-graphics = "0.8.1"
```

## 导入模块

添加以下必需的导入：

```rust

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
```

## 样板代码：初始化 I2C 和显示屏实例

这部分我们已经在上一章[讲解过](../hello-rust/index.md)。

```rust
let i2c_bus = esp_hal::i2c::master::I2c::new(
    peripherals.I2C0,
    esp_hal::i2c::master::Config::default().with_frequency(Rate::from_khz(400)),
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
```

## 绘制图像

我们创建了一个字节数组常量来表示欧姆符号。

```rust
// 8x5 像素
#[rustfmt::skip]
const IMG_DATA: &[u8] = &[
    0b00111000,
    0b01000100,
    0b01000100,
    0b00101000,
    0b11101110,
];
```

我们将使用 `ImageRaw::new` 函数创建原始图像。需要使用涡轮鱼语法 `::<>` 指定图像宽度（即 8）和像素颜色格式。图像的高度将根据数据长度和格式自动计算。由于我们使用的显示屏模块只有两种颜色，因此将使用 `BinaryColor` 枚举。

然后，我们将在显示屏的起始位置（即 Point zero，x = 0，y = 0）绘制图像。最后，我们将数据刷新到显示模块。

```rust
let raw_image = ImageRaw::<BinaryColor>::new(IMG_DATA, 8);

let image = Image::new(&raw_image, Point::zero());

image.draw(&mut display).unwrap();
display.flush().await.unwrap();
```

## 克隆已有项目

你也可以克隆（或参考）我创建的项目，并导航到 `oled-image` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/oled-image
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

// 8x5 像素
#[rustfmt::skip]
const IMG_DATA: &[u8] = &[
    0b00111000,
    0b01000100,
    0b01000100,
    0b00101000,
    0b11101110,
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

    let raw_image = ImageRaw::<BinaryColor>::new(IMG_DATA, 8);

    let image = Image::new(&raw_image, Point::zero());

    image.draw(&mut display).unwrap();
    display.flush().await.unwrap();

    loop {
        Timer::after(Duration::from_secs(1)).await;
    }
}
```

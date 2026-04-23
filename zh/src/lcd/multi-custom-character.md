## 多格自定义字符（Multi Custom Character）

在本练习中，我们将显示借助多格生成器生成的 Ferris 图像。这里我不会重复项目创建步骤。你可以按照上一章的步骤操作，或者直接克隆上一章的项目并在此基础上工作。

我使用上一页的生成器，用 6 个相邻的网格创建了 Ferris 图像。以下是使用这些字节数组的 Rust 代码。

<img style="display: block; margin: auto;width:400px;" alt="custom characters grid" src="./images/ferris-with-6-grids-on-lcd-display.png"/>

### 字符生成的字节数组

```rust
const SYMBOL1: [u8; 8] = [
    0b00110, 0b01000, 0b01110, 0b01000, 0b00100, 0b00011, 0b00100, 0b01000,
];

const SYMBOL2: [u8; 8] = [
    0b00000, 0b00000, 0b00000, 0b10001, 0b10001, 0b11111, 0b00000, 0b00000,
];

const SYMBOL3: [u8; 8] = [
    0b01100, 0b00010, 0b01110, 0b00010, 0b00100, 0b11000, 0b00100, 0b00010,
];

const SYMBOL4: [u8; 8] = [
    0b01000, 0b01000, 0b00100, 0b00011, 0b00001, 0b00010, 0b00101, 0b01000,
];

const SYMBOL5: [u8; 8] = [
    0b00000, 0b00000, 0b00000, 0b11111, 0b01010, 0b10001, 0b00000, 0b00000,
];

const SYMBOL6: [u8; 8] = [
    0b00010, 0b00010, 0b00100, 0b11000, 0b10000, 0b01000, 0b10100, 0b00010,
];
```

### 将它们声明为字符

```rust
lcd.custom_char(&mut Delay, &SYMBOL1, 0);
lcd.custom_char(&mut Delay, &SYMBOL2, 1);
lcd.custom_char(&mut Delay, &SYMBOL3, 2);
lcd.custom_char(&mut Delay, &SYMBOL4, 3);
lcd.custom_char(&mut Delay, &SYMBOL5, 4);
lcd.custom_char(&mut Delay, &SYMBOL6, 5);
```

### 显示

让我们将前 3 个网格写入 LCD 显示屏的第一行，然后将后 3 个写入第二行。

```rust
lcd.set_cursor(&mut Delay, 0, 4)
    .write(&mut Delay, CustomChar(0))
    .write(&mut Delay, CustomChar(1))
    .write(&mut Delay, CustomChar(2));

lcd.set_cursor(&mut Delay, 1, 4)
    .write(&mut Delay, CustomChar(3))
    .write(&mut Delay, CustomChar(4))
    .write(&mut Delay, CustomChar(5));
```


## 克隆现有项目

你可以克隆（或参考）我创建的项目并导航到 `lcd-custom-multi` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/lcd-custom-multi/
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
use embassy_time::Delay;
use embassy_time::{Duration, Timer};
use esp_hal::clock::CpuClock;
use esp_hal::timer::timg::TimerGroup;
use esp_println as _;

// I2C
use esp_hal::i2c::master::Config as I2cConfig; // for convenience, importing as alias
use esp_hal::i2c::master::I2c;
use esp_hal::time::Rate;

// HD44780 Driver
use liquid_crystal::I2C;
use liquid_crystal::LiquidCrystal;
use liquid_crystal::prelude::*;

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

// 这将创建一个 esp-idf 引导加载程序所需的默认应用描述符。
// 更多信息请参阅：<https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
// This creates a default app-descriptor required by the esp-idf bootloader.
// For more information see: <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
esp_bootloader_esp_idf::esp_app_desc!();

const SYMBOL1: [u8; 8] = [
    0b00110, 0b01000, 0b01110, 0b01000, 0b00100, 0b00011, 0b00100, 0b01000,
];

const SYMBOL2: [u8; 8] = [
    0b00000, 0b00000, 0b00000, 0b10001, 0b10001, 0b11111, 0b00000, 0b00000,
];

const SYMBOL3: [u8; 8] = [
    0b01100, 0b00010, 0b01110, 0b00010, 0b00100, 0b11000, 0b00100, 0b00010,
];

const SYMBOL4: [u8; 8] = [
    0b01000, 0b01000, 0b00100, 0b00011, 0b00001, 0b00010, 0b00101, 0b01000,
];

const SYMBOL5: [u8; 8] = [
    0b00000, 0b00000, 0b00000, 0b11111, 0b01010, 0b10001, 0b00000, 0b00000,
];

const SYMBOL6: [u8; 8] = [
    0b00010, 0b00010, 0b00100, 0b11000, 0b10000, 0b01000, 0b10100, 0b00010,
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
        // I2cConfig is alias of esp_hal::i2c::master::I2c::Config
        I2cConfig::default().with_frequency(Rate::from_khz(400)),
    )
    .unwrap()
    .with_scl(peripherals.GPIO18)
    .with_sda(peripherals.GPIO23)
    .into_async();

    let mut i2c_interface = I2C::new(i2c_bus, 0x27);
    let mut lcd = LiquidCrystal::new(&mut i2c_interface, Bus4Bits, LCD16X2);
    lcd.begin(&mut Delay);

    // 定义字符
    // Define the characters
    lcd.custom_char(&mut Delay, &SYMBOL1, 0);
    lcd.custom_char(&mut Delay, &SYMBOL2, 1);
    lcd.custom_char(&mut Delay, &SYMBOL3, 2);
    lcd.custom_char(&mut Delay, &SYMBOL4, 3);
    lcd.custom_char(&mut Delay, &SYMBOL5, 4);
    lcd.custom_char(&mut Delay, &SYMBOL6, 5);

    lcd.set_cursor(&mut Delay, 0, 4)
        .write(&mut Delay, CustomChar(0))
        .write(&mut Delay, CustomChar(1))
        .write(&mut Delay, CustomChar(2));

    lcd.set_cursor(&mut Delay, 1, 4)
        .write(&mut Delay, CustomChar(3))
        .write(&mut Delay, CustomChar(4))
        .write(&mut Delay, CustomChar(5));

    loop {
        Timer::after(Duration::from_secs(1)).await;
    }
}
```

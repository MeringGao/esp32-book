## 在 LCD 显示屏上显示 Ferris

让我们用单个字符创建 Ferris（我试着让它看起来像螃蟹；如果你有更好的设计，欢迎发送拉取请求 pull request）。事实上，我们可以组合 4 个或 6 个相邻的网格来显示单个符号。创意由你发挥，你可以随意改进它。

我们将使用上一页的自定义字符生成器来创建这个符号。这将给我们提供可以使用的字节数组。

<img style="display: block; margin: auto;" alt="lcd1602" src="./images/custom-character-ferris.png"/>

请注意，之前的 hd44780-driver crate 不支持自定义字符（在撰写本章时）。为了解决这个问题，我们可以使用 liquid_crystal crate，它允许我们处理自定义字符。

## 使用 esp-generate 生成项目

我们将为此项目启用异步（async，Embassy）支持。要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 lcd-custom
```

这将打开一个屏幕，要求你选择选项。

- 首先，选择 "Enable unstable HAL features."。只有在此之后，你才能选择 embassy 框架支持
- 选择 "Adds embassy framework support."

只需按键盘上的 "s" 保存即可。

## 更新 Cargo.toml

```toml
liquid_crystal = "0.2.0"
```

## 导入（Imports）

首先添加必要的导入：

```rust
// I2C
use esp_hal::i2c::master::Config as I2cConfig; // for convenience, importing as alias
use esp_hal::i2c::master::I2c;
use esp_hal::time::Rate;

// HD44780 Driver
use liquid_crystal::prelude::*;
use liquid_crystal::LiquidCrystal;
use liquid_crystal::I2C;
```

### 初始化 I2C

```rust
let i2c_bus = I2c::new(
        peripherals.I2C0,
        // I2cConfig is alias of esp_hal::i2c::master::I2c::Config
        I2cConfig::default().with_frequency(Rate::from_khz(400)),
    )
    .unwrap()
    .with_scl(peripherals.GPIO18)
    .with_sda(peripherals.GPIO23)
    .into_async();
```

### 初始化 LCD 接口

```rust
let mut i2c_interface = I2C::new(i2c_bus, 0x27);
let mut lcd = LiquidCrystal::new(&mut i2c_interface, Bus4Bits, LCD16X2);
lcd.begin(&mut Delay);
```

### 我们为自定义字符生成的字节数组

```rust
const FERRIS: [u8; 8] = [
    0b01010, 0b10001, 0b10001, 0b01110, 0b01110, 0b01110, 0b11111, 0b10001,
];
// 定义字符
// Define the character
lcd.custom_char(&mut Delay, &FERRIS, 0);
```

### 显示字符

显示字符很简单。你只需要使用 CustomChar 枚举并传递自定义字符的索引。我们只定义了一个自定义字符，它位于位置 0。

```rust
lcd.write(&mut Delay, CustomChar(0));
// 普通文字
// normal text
lcd.write(&mut Delay, Text(" implRust!"));
```

## 克隆现有项目

你可以克隆（或参考）我创建的项目并导航到 `lcd-custom` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/lcd-custom/
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

const FERRIS: [u8; 8] = [
    0b01010, 0b10001, 0b10001, 0b01110, 0b01110, 0b01110, 0b11111, 0b10001,
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
    // Define the character
    lcd.custom_char(&mut Delay, &FERRIS, 0);

    lcd.write(&mut Delay, CustomChar(0));
    lcd.write(&mut Delay, Text(" implRust!"));

    loop {
        info!("Hello world!");
        Timer::after(Duration::from_secs(1)).await;
    }
}
````

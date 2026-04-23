# 在 LCD 显示屏上显示 "Hello, Rust!"

在这个程序中，我们将在 LCD 显示屏上打印 "Hello, Rust!" 文字。

## HD44780 驱动（Drivers）

在研究过程中，我发现了许多用于控制 LCD 显示屏的 Rust crate，但以下两个表现突出。在这个程序中，我们将首先使用 `hd44780-driver` crate。

- [hd44780-driver](https://crates.io/crates/hd44780-driver)
- [liquid_crystal](https://crates.io/crates/liquid_crystal)

## 使用 esp-generate 生成项目

我们将为此项目启用异步（async，Embassy）支持。要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 lcd-hello
```

这将打开一个屏幕，要求你选择选项。

- 首先，选择 "Enable unstable HAL features."。只有在此之后，你才能选择 embassy 框架支持
- 选择 "Adds embassy framework support."

只需按键盘上的 "s" 保存即可。

## 更新 Cargo.toml

我们将使用 hd44780-driver crate；并直接从 GitHub 仓库使用最新版本，使用 rev 属性指定 Git 提交。要启用异步支持，我们可以指定 "embedded-hal-async" 特性。

```toml
# hd44780-driver = "0.4.0"
hd44780-driver = { git = "https://github.com/JohnDoneth/hd44780-driver", rev = "9009f2c24771ba0a20f8f7534471c9869188f76c", features = [
    "embedded-hal-async",
] }
```

## 导入（Imports）

首先添加必要的导入：

```rust
// I2C
use esp_hal::i2c::master::Config as I2cConfig; // for convenience, importing as alias
use esp_hal::i2c::master::I2c;
use esp_hal::time::Rate;

// HD44780 Driver
use hd44780_driver::HD44780;
use hd44780_driver::memory_map::MemoryMap1602;
use hd44780_driver::setup::DisplayOptionsI2C;
```

### 初始化 I2C

我们已配置 I2C 接口以将 ESP32 与 LCD 模块连接。我们将 I2C 总线设置为以 400 kHz 运行。我们将 GPIO18 分配给 SCL（Serial Clock Line，串行时钟线），将 GPIO23 分配给 SDA（Serial Data Line，串行数据线）。我们还启用了异步操作。

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

### 初始化显示屏

我们需要指定 LCD 模块的 I2C 地址（address）。该地址通常在模块的规格书中提供。最常见的是 0x27。最好通过数据手册（datasheet）确认，或者通过扫描 I2C 设备来确认。

```rust
let i2c_address = 0x27;

let Ok(mut lcd) = HD44780::new(
    DisplayOptionsI2C::new(MemoryMap1602::new()).with_i2c_bus(i2c_bus, i2c_address),
    &mut embassy_time::Delay,
) else {
    panic!("failed to initialize display");
};
```

### 准备显示屏

让我们将显示屏重置为默认状态，将光标移到开头，并清除显示屏上任何现有字符。

```rust
// Unshift display and set cursor to 0
lcd.reset(&mut embassy_time::Delay).unwrap();

// Clear existing characters
lcd.clear(&mut embassy_time::Delay).unwrap();
```

### 向 LCD 写入文字

让我们在 LCD 模块的两行上都写入文字。

```rust

// Display the following string
lcd.write_str("impl Rust", &mut embassy_time::Delay)
    .unwrap();

// Move the cursor to the second line
lcd.set_cursor_xy((0, 1), &mut embassy_time::Delay).unwrap();

// Display the following string on the second line
lcd.write_str("Hello, Ferris!", &mut embassy_time::Delay)
    .unwrap();
```

## 克隆现有项目

你可以克隆（或参考）我创建的项目并导航到 `lcd-hello` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/lcd-hello/
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
use esp_hal::i2c::master::Config as I2cConfig; // for convenience, importing as alias
use esp_hal::i2c::master::I2c;
use esp_hal::time::Rate;

// HD44780 Driver
use hd44780_driver::HD44780;
use hd44780_driver::memory_map::MemoryMap1602;
use hd44780_driver::setup::DisplayOptionsI2C;

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

    let i2c_address = 0x27;

    let Ok(mut lcd) = HD44780::new(
        DisplayOptionsI2C::new(MemoryMap1602::new()).with_i2c_bus(i2c_bus, i2c_address),
        &mut embassy_time::Delay,
    ) else {
        panic!("failed to initialize display");
    };

    // 取消显示屏移位并将光标设为 0
    // Unshift display and set cursor to 0
    lcd.reset(&mut embassy_time::Delay).unwrap();

    // 清除现有字符
    // Clear existing characters
    lcd.clear(&mut embassy_time::Delay).unwrap();

    // 显示以下字符串
    // Display the following string
    lcd.write_str("impl Rust", &mut embassy_time::Delay)
        .unwrap();

    // 将光标移到第二行
    // Move the cursor to the second line
    lcd.set_cursor_xy((0, 1), &mut embassy_time::Delay).unwrap();

    // 在第二行显示以下字符串
    // Display the following string on the second line
    lcd.write_str("Hello, Ferris!", &mut embassy_time::Delay)
        .unwrap();

    loop {
        info!("Hello world!");
        Timer::after(Duration::from_secs(1)).await;
    }
}
```

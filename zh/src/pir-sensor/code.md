# 使用 Rust 编写 PIR 传感器运动检测代码

让我们编写一个简单的程序，每当检测到运动时打印一条消息。这将帮助我们微调 PIR 传感器的设置并掌握一些基本概念。完成后，我们将构建一个完整的防盗报警模拟系统，使用蜂鸣器和板载 LED（或外接 LED，你可以根据需要调整）来让它更有趣。

### 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 pir-sensor
```

这将弹出一个屏幕，要求你选择选项。

在这个练习中，我们将向系统控制台打印消息。为了做到这一点，我们需要启用日志功能。

所以，滚动到 "Flashing, logging and debugging (espflash)" 并按回车键。

选择 "Use defmt to print messages"。

直接按键盘上的 "s" 保存即可。

## 传感器输出引脚连接到 ESP32 的输入引脚

我们将 GPIO 33 配置为输入引脚，初始状态为下拉（Pull Down）。该引脚连接到 PIR 传感器的输出，当检测到运动时，输出会变为高电平。

```rust
let sensor_pin = Input::new(
    peripherals.GPIO33,
    InputConfig::default().with_pull(Pull::Down),
);
```

## 逻辑

思路很简单：我们在循环中持续检查传感器的输出。当传感器的输出变为高电平时，我们打印消息 "Motion detected" 并添加一个短暂的延时。

```rust
loop {
    if sensor_pin.is_high() {
        info!("Motion detected");
        blocking_delay(Duration::from_millis(100));
    }
    blocking_delay(Duration::from_millis(100));
}
```

## 克隆现有项目

你可以克隆（或参考）我创建的项目，并导航到 `pir-sensor` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/pir-sensor
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
use esp_hal::gpio::{Input, InputConfig, Pull};
use esp_hal::main;
use esp_hal::time::{Duration, Instant};
use esp_println as _;

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

// 创建一个 ESP-IDF 引导加载程序所需的默认应用描述符。
// This creates a default app-descriptor required by the esp-idf bootloader.
// 更多信息请参阅：<https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
// For more information see: <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
esp_bootloader_esp_idf::esp_app_desc!();

#[main]
fn main() -> ! {
    // generator version: 1.0.0

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    let sensor_pin = Input::new(
        peripherals.GPIO33,
        InputConfig::default().with_pull(Pull::Down),
    );

    loop {
        if sensor_pin.is_high() {
            info!("Motion detected");
            blocking_delay(Duration::from_millis(100));
        }
        blocking_delay(Duration::from_millis(100));
    }
}

fn blocking_delay(duration: Duration) {
    let delay_start = Instant::now();
    while delay_start.elapsed() < duration {}
}
```

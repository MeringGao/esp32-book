# 使用 Rust 和 ESP32 通过光敏电阻在黑暗中点亮 LED

### 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 ldr-dracula
```

这将打开一个屏幕，要求你选择选项。在最新的 esp-hal 中，ADC 需要我们显式启用不稳定特性（unstable features）。

- 因此请选择 "Enable unstable HAL features."

在这个练习中，我们将向系统控制台打印消息。为了做到这一点，我们需要启用日志功能。

- 所以，滚动到 "Flashing, logging and debugging (espflash)" 并按回车键。

- 然后，选择 "Use defmt to print messages"。


然后按键盘上的 "s" 保存。

## 更新 cargo.toml

nb crate 通过返回 nb::Result 并在操作未就绪时返回 WouldBlock 错误来简化非阻塞 I/O（例如，读取传感器、UART 数据），允许你稍后重试而不会阻塞。

如果你想知道为什么需要它，ADC 的 read_oneshot 函数返回一个 nb::Result，如果尚未就绪，可能会返回 nb::Error::WouldBlock。用 nb::block 包装它会使你的代码不断重试，直到 ADC 完成其工作并返回一个正确的结果。

```toml
nb = "1.1.0"
```

### 设置 LED

我们之前已经做过这个；只需将连接到 LED 的 GPIO 33 设置为输出引脚，并将其初始化为低电平状态。

```rust
let mut led = Output::new(peripherals.GPIO33, Level::Low, OutputConfig::default());
```

### 配置 ADC

我们将 GPIO 4 配置为 ADC 输入引脚，它是 ADC2 的通道之一。我们将应用 11dB 的衰减级别，使 ADC 能够测量 150 mV 到约 2450 mV 的输入电压范围。

```rust
let adc_pin = peripherals.GPIO4;
let mut adc2_config = AdcConfig::new();
let mut pin = adc2_config.enable_pin(adc_pin, Attenuation::_11dB);
let mut adc2 = Adc::new(peripherals.ADC2, adc2_config);
```

### 单次读取（Oneshot read）

`read_oneshot` 函数在指定引脚上启动单次 ADC 转换。它是非阻塞的，并返回一个包装在 Result 中的 16 位值。但是，我们将使用 `nb::block` 来阻塞直到转换完成，然后继续执行。

```rust
let pin_value: u16 = nb::block!(adc2.read_oneshot(&mut pin)).unwrap();
```

### 切换 LED

一旦我们得到数字值，打开或关闭 LED 就是简单的逻辑。如果引脚值大于 3500（你可以根据需要调整此阈值），我们将 LED 设为高电平以打开它。否则，我们将关闭 LED。当房间变暗且 LDR 的阻值（R2）较高时，通常会达到 3500 这个阈值，从而产生更高的电压。

```rust
if pin_value > 3500 {
    led.set_high();
} else {
    led.set_low();
}
```

如果你想知道为什么 LDR 的高阻值会导致更高的电压，我建议重新阅读["工作原理"](../how-it-works.md)部分，并在仿真器中进行实验。

### 最终代码

```rust
#![no_std]
#![no_main]
#![deny(
    clippy::mem_forget,
    reason = "mem::forget is generally not safe to do with esp_hal types, especially those \
    holding buffers for the duration of a data transfer."
)]

use esp_hal::clock::CpuClock;
use esp_hal::delay::Delay;
use esp_hal::gpio::{Level, Output, OutputConfig};
use esp_hal::main;
use esp_println as _;

// ADC
use esp_hal::analog::adc::{Adc, AdcConfig, Attenuation};

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

    let mut led = Output::new(peripherals.GPIO33, Level::Low, OutputConfig::default());

    let adc_pin = peripherals.GPIO4;
    let mut adc2_config = AdcConfig::new();
    let mut pin = adc2_config.enable_pin(adc_pin, Attenuation::_11dB);
    let mut adc2 = Adc::new(peripherals.ADC2, adc2_config);
    let delay = Delay::new();

    loop {
        let pin_value: u16 = nb::block!(adc2.read_oneshot(&mut pin)).unwrap();
        esp_println::println!("{}", pin_value);

        if pin_value > 3500 {
            led.set_high();
        } else {
            led.set_low();
        }

        delay.delay_millis(500);
    }
}
```

## 克隆现有项目

你也可以克隆（或参考）我创建的项目，并导航到 `ldr-dracula` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/ldr-dracula
```

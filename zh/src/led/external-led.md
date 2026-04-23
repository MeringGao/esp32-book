# 使用外接 LED（External LED）

你可以用外接 LED（external LED）实现同样的渐变效果。

## 硬件需求

- 外接 LED（External LED）
- 电阻（Resistor）(330 欧姆)
- 跳线（Jumper wires）(可选)
- 面包板（Breadboard）(可选) - 你可能需要两个面包板才能正确放置 ESP32 开发板，因为它相当宽。我买了两个小面包板，将 ESP32 的一侧放在每个面包板上。


## 电路
- 将外接 LED 的阳极（anode，较长的引脚）通过 330 欧姆电阻连接到 ESP32 的 GPIO 5
- 将 LED 的阴极（cathode，较短的引脚）连接到 ESP32 的地线（GND）引脚
<br/><br/>
<img style="display: block; margin: auto;" src="./images/esp32-external-led-circuit.jpg"/>


## 代码修改

在代码中，你只需要将 GPIO 编号从 2 改为 5。

```rust
let led = peripherals.GPIO5;
```

## 高速通道

只改一行代码没什么意思。这次让我们使用高速通道（high-speed channel）。要做到这一点，我们必须传入 `HighSpeed` 结构体，并更新时钟源以使用 `HSClockSource` 枚举。

```rust
let ledc = Ledc::new(peripherals.LEDC);

let mut hstimer0 = ledc.timer::<HighSpeed>(timer::Number::Timer0);
hstimer0
    .configure(timer::config::Config {
        duty: timer::config::Duty::Duty5Bit,
        clock_source: timer::HSClockSource::APBClk,
        frequency: Rate::from_khz(24),
    })
    .unwrap();
```


## 克隆现有项目
你也可以克隆（或参考）我创建的项目，并导航到 `led-highfader` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/led-highfader
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

use esp_hal::clock::CpuClock;
use esp_hal::gpio::DriveMode;
use esp_hal::main;
use esp_hal::time::Rate;

// 用于 LEDC
// For LEDC
use esp_hal::ledc::channel::ChannelIFace;
use esp_hal::ledc::timer::TimerIFace;
use esp_hal::ledc::{HighSpeed, Ledc, channel, timer};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

// 这会创建一个 ESP-IDF 引导加载程序所需的默认应用描述符。
// 更多信息请参阅：https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description
// This creates a default app-descriptor required by the esp-idf bootloader.
// For more information see: <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
esp_bootloader_esp_idf::esp_app_desc!();

#[main]
fn main() -> ! {
    // 生成器版本：1.0.0
    // generator version: 1.0.0

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    // let led = peripherals.GPIO2;
    let led = peripherals.GPIO5;

    let ledc = Ledc::new(peripherals.LEDC);

    let mut hstimer0 = ledc.timer::<HighSpeed>(timer::Number::Timer0);
    hstimer0
        .configure(timer::config::Config {
            duty: timer::config::Duty::Duty5Bit,
            clock_source: timer::HSClockSource::APBClk,
            frequency: Rate::from_khz(24),
        })
        .unwrap();

    let mut channel0 = ledc.channel(channel::Number::Channel0, led);
    channel0
        .configure(channel::config::Config {
            timer: &hstimer0,
            duty_pct: 10,
            drive_mode: DriveMode::PushPull,
        })
        .unwrap();

    loop {
        channel0.start_duty_fade(0, 100, 1000).unwrap();
        while channel0.is_duty_fade_running() {}
        channel0.start_duty_fade(100, 0, 1000).unwrap();
        while channel0.is_duty_fade_running() {}
    }
}
```

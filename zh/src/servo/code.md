# 使用 Rust 编写代码让舵机与 ESP32 平滑旋转

## 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 servo-motor
```

这将打开一个屏幕，要求你选择选项。在最新的 esp-hal 中，ledc 需要我们显式启用不稳定特性（unstable features）。

- 因此请选择 "Enable unstable HAL features."

然后按键盘上的 "s" 保存。

## 更新依赖

将以下 crate 添加到 Cargo.toml 文件中
```toml
embedded-hal = "1.0.0"
```

然后在 main.rs 文件中导入 SetDutyCycle trait。如果我们想使用 max_duty_cycle、set_duty_cycle 函数，这是必需的
```rust
use embedded_hal::pwm::SetDutyCycle;
```

## 定时器和 PWM 通道

让我们用 50Hz 频率和 12 位分辨率初始化定时器，并配置通道。

```rust
let mut servo = peripherals.GPIO33;
let ledc = Ledc::new(peripherals.LEDC);

let mut hstimer0 = ledc.timer::<HighSpeed>(timer::Number::Timer0);
hstimer0
    .configure(timer::config::Config {
        duty: timer::config::Duty::Duty12Bit,
        clock_source: timer::HSClockSource::APBClk,
        frequency: Rate::from_hz(50),
    })
    .unwrap();

let mut channel0 = ledc.channel(channel::Number::Channel0, servo.reborrow());
channel0
    .configure(channel::config::Config {
        timer: &hstimer0,
        duty_pct: 10,
        drive_mode: DriveMode::PushPull,
    })
    .unwrap();
```

## 辅助函数

我们在上一章中已经解释了这段代码。

```rust
    let max_duty_cycle = channel0.max_duty_cycle() as u32;

    // 最小占空比（2.5%）
    // Minimum duty (2.5%)
    // 对于 12 位 -> 25 * 4096 /1000 => ~ 102
    // For 12bit -> 25 * 4096 /1000 => ~ 102
    let min_duty = (25 * max_duty_cycle) / 1000;
    // 最大占空比（12.5%）
    // Maximum duty (12.5%)
    // 对于 12 位 -> 125 * 4096 /1000 => 512
    // For 12bit -> 125 * 4096 /1000 => 512
    let max_duty = (125 * max_duty_cycle) / 1000;
    // 512 - 102 => 410
    let duty_gap = max_duty - min_duty;
    
    fn duty_from_angle(deg: u32, min_duty: u32, duty_gap: u32) -> u16 {
        let duty = min_duty + ((deg * duty_gap) / 180);
        duty as u16
    }
```

## 旋转

在主循环中，我们首先从 0 度旋转到 180 度。我们添加 10 毫秒的间隔以让它到达位置，这足够了，因为我们是以较小的步进移动的。然后，我们使用 rev() 函数反转范围，使其从 180 度回到 0 度。

```rust
loop {
    for deg in 0..=180 {
        let duty = duty_from_angle(deg, min_duty, duty_gap);
        channel0.set_duty_cycle(duty).unwrap();
        delay.delay_millis(10);
    }
    delay.delay_millis(500);

    for deg in (0..=180).rev() {
        let duty = duty_from_angle(deg, min_duty, duty_gap);
        channel0.set_duty_cycle(duty).unwrap();
        delay.delay_millis(10);
    }
    delay.delay_millis(500);
}
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
use esp_hal::delay::Delay;
use esp_hal::main;

// LEDC
use esp_hal::gpio::DriveMode;
use esp_hal::ledc::channel::ChannelIFace;
use esp_hal::ledc::timer::TimerIFace;
use esp_hal::ledc::{HighSpeed, Ledc, channel, timer};
use esp_hal::time::Rate;

use embedded_hal::pwm::SetDutyCycle;

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

    let mut servo = peripherals.GPIO33;
    let ledc = Ledc::new(peripherals.LEDC);

    let mut hstimer0 = ledc.timer::<HighSpeed>(timer::Number::Timer0);
    hstimer0
        .configure(timer::config::Config {
            duty: timer::config::Duty::Duty12Bit,
            clock_source: timer::HSClockSource::APBClk,
            frequency: Rate::from_hz(50),
        })
        .unwrap();

    let mut channel0 = ledc.channel(channel::Number::Channel0, servo.reborrow());
    channel0
        .configure(channel::config::Config {
            timer: &hstimer0,
            duty_pct: 10,
            drive_mode: DriveMode::PushPull,
        })
        .unwrap();

    let delay = Delay::new();

    let max_duty_cycle = channel0.max_duty_cycle() as u32;

    // 最小占空比（2.5%）
    // Minimum duty (2.5%)
    // 对于 12 位 -> 25 * 4096 /1000 => ~ 102
    // For 12bit -> 25 * 4096 /1000 => ~ 102
    let min_duty = (25 * max_duty_cycle) / 1000;
    // 最大占空比（12.5%）
    // Maximum duty (12.5%)
    // 对于 12 位 -> 125 * 4096 /1000 => 512
    // For 12bit -> 125 * 4096 /1000 => 512
    let max_duty = (125 * max_duty_cycle) / 1000;
    // 512 - 102 => 410
    let duty_gap = max_duty - min_duty;

    loop {
        for deg in 0..=180 {
            let duty = duty_from_angle(deg, min_duty, duty_gap);
            channel0.set_duty_cycle(duty).unwrap();
            delay.delay_millis(10);
        }
        delay.delay_millis(500);

        for deg in (0..=180).rev() {
            let duty = duty_from_angle(deg, min_duty, duty_gap);
            channel0.set_duty_cycle(duty).unwrap();
            delay.delay_millis(10);
        }
        delay.delay_millis(500);
    }
}

fn duty_from_angle(deg: u32, min_duty: u32, duty_gap: u32) -> u16 {
    let duty = min_duty + ((deg * duty_gap) / 180);
    duty as u16
}
```


## 克隆现有项目

你可以克隆（或参考）我创建的项目，并导航到 `servo-motor` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/servo-motor
```

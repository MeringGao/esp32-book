## 使用 Rust 编写 HC-SR04 超声波传感器与 ESP32 的代码

我们将从使用模板生成项目开始，然后修改代码以满足当前项目的需求。

### 使用 esp-generate 生成项目

这一步你在快速入门部分已经完成过了。

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 ultrasonic
```

这将打开一个屏幕，要求你选择选项。在最新的 esp-hal 中，ledc 需要我们显式启用不稳定特性（unstable features）。

- 因此请选择 "Enable unstable HAL features."

然后按键盘上的 "s" 保存。

## 设置 LED 引脚并配置 PWM

你现在应该已经理解这段代码了。如果还不理解，请先完成 LED 的[渐变章节](../led/index.md)。

快速回顾：在这里，我们为 LED 配置 PWM，通过调整占空比（Duty Cycle）来控制亮度。

```rust
// let led = peripherals.GPIO2; // 使用板载 LED
let led = peripherals.GPIO33;

// 配置 LEDC
let mut ledc = Ledc::new(peripherals.LEDC);
ledc.set_global_slow_clock(LSGlobalClkSource::APBClk);

let mut lstimer0 = ledc.timer::<LowSpeed>(timer::Number::Timer0);
lstimer0
    .configure(timer::config::Config {
        duty: timer::config::Duty::Duty5Bit,
        clock_source: timer::LSClockSource::APBClk,
        frequency: Rate::from_khz(24),
    })
    .unwrap();

let mut channel0 = ledc.channel(channel::Number::Channel0, led);
channel0
    .configure(channel::config::Config {
        timer: &lstimer0,
        duty_pct: 10,
        drive_mode: DriveMode::PushPull,
    })
    .unwrap();
```

## 设置触发引脚（Trigger Pin）

我们将 GPIO 5 配置为输出引脚，初始状态设为低电平（LOW）。如果你想知道为什么它是输出，那是因为我们从 ESP32 向超声波模块发送信号。该引脚连接到超声波模块的 Trig 引脚。

```rust
let mut trig = Output::new(peripherals.GPIO5, Level::Low, OutputConfig::default());
```

## 设置回响引脚（Echo Pin）

我们将 GPIO 18 配置为输入引脚，因为超声波模块会将信号传回 ESP32。该引脚的初始状态将设为下拉（Pull Down），以确保它从低电平开始。

```rust
let echo = Input::new(
    peripherals.GPIO18,
    InputConfig::default().with_pull(Pull::Down),
);
```

## 🦇 点亮它

### 第一步：发送触发脉冲

我们将 trig 引脚设为低电平，以便重新开始。然后将 trig 引脚置为高电平 10 微秒，再恢复为低电平。这将触发模块发送超声波。

```rust
// 确保触发引脚在开始之前为低电平
// Ensure the Trigger pin is low before starting
trig.set_low();
delay.delay_micros(2);

// 发送 10 微秒的高电平脉冲
// Send a 10-microseconds high pulse
trig.set_high();
delay.delay_micros(10);
trig.set_low();
```

### 第二步：测量脉冲宽度

接下来，我们将使用两个循环。第一个循环会在 echo 引脚状态为低电平时持续运行。一旦它变为高电平，我们就会将当前时间记录到一个变量中。然后，我们启动第二个循环，只要 echo 引脚保持高电平，它就会继续运行。当它恢复为低电平时，我们将当前时间记录到另一个变量中。这两个时间之差就是脉冲宽度。

```rust
 // 测量信号保持高电平的持续时间
 // Measure the duration the signal remains high
while echo.is_low() {}
let time1 = rtc.current_time_us();
while echo.is_high() {}
let time2 = rtc.current_time_us();
let pulse_width = time2 - time1;
```

### 第三步：计算距离

要计算距离，我们需要使用脉冲宽度。脉冲宽度告诉我们超声波传播到障碍物并返回所需的时间。由于脉冲代表往返时间，我们将其除以 2 以计算单程距离。

空气中声速约为每微秒 0.0343 厘米。将时间（以微秒为单位）乘以该值再除以 2，即可得到以厘米为单位的障碍物距离。

```rust
let distance = (pulse_width * 0.0343) / 2.0;
```

### 第四步：LED 的 PWM 占空比

最后，我们根据测量的距离调整 LED 亮度。

占空比百分比使用我们自己的逻辑计算，你可以根据需要修改它。当物体距离小于 30 厘米时，LED 亮度会增加。物体离超声波模块越近，计算出的比率就越高，从而调整占空比。这会导致 LED 亮度随着物体靠近传感器而逐渐增加。

```rust
let duty_pct: u8 = if distance < 30.0 {
    let ratio = (30.0 - distance) / 30.0;
    let p = (ratio * 100.0) as u8;
    p.min(100)
} else {
    0
};

if let Err(e) = channel0.set_duty(duty_pct) {
    esp_println::println!("Failed to set duty cycle: {:?}", e);
}
```

### 完整代码

```rust
#![no_std]
#![no_main]
#![deny(
    clippy::mem_forget,
    reason = "mem::forget is generally not safe to do with esp_hal types, especially those \
    holding buffers for the duration of a data transfer."
)]

use esp_hal::clock::CpuClock;
use esp_hal::main;

// LEDC
use esp_hal::gpio::DriveMode;
use esp_hal::gpio::{InputConfig, OutputConfig};
use esp_hal::ledc::{LSGlobalClkSource, LowSpeed};
use esp_hal::time::Rate;

use esp_hal::{
    delay::Delay,
    gpio::{Input, Level, Output, Pull},
    ledc::{
        Ledc,
        channel::{self, ChannelIFace},
        timer::{self, TimerIFace},
    },
    rtc_cntl::Rtc,
};

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

    // let led = peripherals.GPIO2; // 使用板载 LED
    // let led = peripherals.GPIO2; // uses onboard LED
    let led = peripherals.GPIO33;

    // 配置 LEDC
    // Configure LEDC
    let mut ledc = Ledc::new(peripherals.LEDC);
    ledc.set_global_slow_clock(LSGlobalClkSource::APBClk);

    let mut lstimer0 = ledc.timer::<LowSpeed>(timer::Number::Timer0);
    lstimer0
        .configure(timer::config::Config {
            duty: timer::config::Duty::Duty5Bit,
            clock_source: timer::LSClockSource::APBClk,
            frequency: Rate::from_khz(24),
        })
        .unwrap();

    let mut channel0 = ledc.channel(channel::Number::Channel0, led);
    channel0
        .configure(channel::config::Config {
            timer: &lstimer0,
            duty_pct: 10,
            drive_mode: DriveMode::PushPull,
        })
        .unwrap();

    // 用于 HC-SR04 超声波传感器
    // For HC-SR04 Ultrasonic
    let mut trig = Output::new(peripherals.GPIO5, Level::Low, OutputConfig::default());
    let echo = Input::new(
        peripherals.GPIO18,
        InputConfig::default().with_pull(Pull::Down),
    );

    let delay = Delay::new(); // 由于我们使用了不稳定特性，可以使用这个
                              // We can use this since we are using unstable features

    let rtc = Rtc::new(peripherals.LPWR);

    loop {
        delay.delay_millis(5);

        // 触发超声波
        // Trigger ultrasonic waves
        trig.set_low();
        delay.delay_micros(2);
        trig.set_high();
        delay.delay_micros(10);
        trig.set_low();

        // 测量信号保持高电平的持续时间
        // Measure the duration the signal remains high
        while echo.is_low() {}
        let time1 = rtc.current_time_us();
        while echo.is_high() {}
        let time2 = rtc.current_time_us();
        let pulse_width = time2 - time1;

        // 从脉冲宽度推导距离
        // Derive distance from the pulse width
        let distance = (pulse_width as f64 * 0.0343) / 2.0;
        // esp_println::println!("Pulse Width: {}", pulse_width);
        // esp_println::println!("Distance: {}", distance);

        // 我们自己计算距离的占空比百分比的逻辑
        // Our own logic to calculate duty cycle percentage for the distance
        let duty_pct: u8 = if distance < 30.0 {
            let ratio = (30.0 - distance) / 30.0;
            let p = (ratio * 100.0) as u8;
            p.min(100)
        } else {
            0
        };

        if let Err(e) = channel0.set_duty(duty_pct) {
            // esp_println::println!("Failed to set duty cycle: {:?}", e);
            panic!("Failed to set duty cycle: {:?}", e);
        }

        delay.delay_millis(60);
    }
}
```

## 克隆现有项目

你可以克隆（或参考）我创建的项目，并导航到 `ultrasonic` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/ultrasonic
```

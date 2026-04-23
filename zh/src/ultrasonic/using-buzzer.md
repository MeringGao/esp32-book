# 使用 ESP32 和超声波传感器实现物体检测蜂鸣器警报

在我们之前的练习中，我们使用了一个 LED，当物体靠近超声波传感器模块时，LED 会变亮。现在，我们将用有源蜂鸣器（Active Buzzer）代替 LED。当物体靠近传感器时，蜂鸣器会发出声音。虽然你可以使用 PWM 来创建不同的声音或音调，但在这个项目中我们会保持简单。当物体靠近传感器时，蜂鸣器会发出蜂鸣声。

## 电路

电路与之前几乎相同。唯一的区别是你需要移除 LED 及其相关电阻。取而代之的是，将蜂鸣器连接到 GPIO 33。我们将蜂鸣器的正极引脚（通常标有加号）连接到 GPIO 33，另一个引脚连接到地线。

<img style="display: block; margin: auto;" alt="hc-sr04 with buzzer and ESP32 circuit" src="./images/ESP32-HC-SR04-circuit-buzzer.png"/>

### 使用 esp-generate 生成项目

这一步你在快速入门部分已经完成过了。

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 ultrasonic-alert
```

这将打开一个屏幕，要求你选择选项。我们不需要不稳定特性或其他任何特性。所以直接按键盘上的 "s" 保存即可。

## 代码

我们将 GPIO 33 设置为输出引脚，初始状态为低电平。这与 LED 代码相同；唯一的改变是变量名。

```rust
let mut buzzer = Output::new(peripherals.GPIO33, Level::Low, OutputConfig::default());
```

我们不需要用于 LED 的定时器或 PWM 配置。相反，如果距离小于 30 厘米，我们将蜂鸣器设为高电平（高电平时它会发出声音）；否则，它将保持低电平。

```rust
if distance < 30.0 {
    buzzer.set_high();
} else {
    buzzer.set_low();
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
use esp_hal::gpio::{Input, InputConfig, Level, Output, OutputConfig, Pull};
use esp_hal::main;
use esp_hal::time::{Duration, Instant};

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

    let mut buzzer = Output::new(peripherals.GPIO33, Level::Low, OutputConfig::default());

    // 用于 HC-SR04 超声波传感器
    // For HC-SR04 Ultrasonic
    let mut trig = Output::new(peripherals.GPIO5, Level::Low, OutputConfig::default());
    let echo = Input::new(
        peripherals.GPIO18,
        InputConfig::default().with_pull(Pull::Down),
    );

    loop {
        blocking_delay(Duration::from_millis(5));

        // 触发超声波
        // Trigger ultrasonic waves
        trig.set_low();
        blocking_delay(Duration::from_micros(2));
        trig.set_high();
        blocking_delay(Duration::from_micros(10));
        trig.set_low();

        // 测量信号保持高电平的持续时间
        // Measure the duration the signal remains high
        while echo.is_low() {}
        let time1 = Instant::now();
        while echo.is_high() {}
        let pulse_width = time1.elapsed().as_micros();

        // 从脉冲宽度推导距离
        // Derive distance from the pulse width
        let distance = (pulse_width as f64 * 0.0343) / 2.0;
        // esp_println::println!("Pulse Width: {}", pulse_width);
        // esp_println::println!("Distance: {}", distance);

        if distance < 30.0 {
            buzzer.set_high();
        } else {
            buzzer.set_low();
        }

        blocking_delay(Duration::from_millis(60));
    }
}

fn blocking_delay(duration: Duration) {
    let delay_start = Instant::now();
    while delay_start.elapsed() < duration {}
}
```

## 克隆现有项目

你可以克隆（或参考）我创建的项目，并导航到 `ultrasonic-alert` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/ultrasonic-alert
```

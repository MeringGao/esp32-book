# 使用 ESP32、PIR 传感器和 Rust 构建简易防盗报警器

让我们制作一个简单的防盗报警器，激活蜂鸣器一小段时间然后关闭它。我们还会打开板载 LED，它连接到 GPIO2。请随意根据你的需求进行调整！

## 硬件需求

- 有源蜂鸣器（Active Buzzer）
- PIR 传感器
- 跳线

## 电路

PIR 传感器的连接与之前相同（参见[电路](./circuit.md)）。我们将传感器的中间输出引脚连接到 GPIO 33。

### 蜂鸣器引脚连接：
<table style="margin-bottom:20px">
  <thead>
    <tr>
      <th>ESP32 引脚</th>
      <th style="width: 250px; margin: 0 auto;">导线</th>
      <th>蜂鸣器引脚</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GPIO 18</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire blue" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>正极引脚</td>
    </tr>
    <tr>
      <td>GND</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire black" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>负极引脚</td>
    </tr>
  </tbody>
</table>

<img style="display: block; margin: auto;" alt="HC-SR501" src="./images/esp32-pir-sensor-burglar-alarm.png"/>

### 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 burglar-alarm
```

## 蜂鸣器和 LED 引脚

我们将 GPIO 18 设置为输出引脚，初始状态为低电平，用于连接有源蜂鸣器。板载 LED 连接到 GPIO 2，也将被配置为输出引脚，初始状态为低电平。

```rust
let mut buzzer_pin = Output::new(peripherals.GPIO18, Level::Low, OutputConfig::default());
let mut led = Output::new(peripherals.GPIO2, Level::Low, OutputConfig::default());
```

## 逻辑

逻辑与之前的代码类似。不过，这次当检测到运动时（即传感器引脚为高电平时），我们不只是打印一条消息，而是将蜂鸣器和 LED 打开一小会儿，然后关闭它们。

```rust
loop {
    if sensor_pin.is_high() {
        buzzer_pin.set_high();
        led.set_high();
        blocking_delay(Duration::from_millis(100));
        buzzer_pin.set_low();
        led.set_low();
    }
    blocking_delay(Duration::from_millis(100));
}
```

## 克隆现有项目

你可以克隆（或参考）我创建的项目，并导航到 `burglar-alarm` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/burglar-alarm
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

    let sensor_pin = Input::new(
        peripherals.GPIO33,
        InputConfig::default().with_pull(Pull::Down),
    );

    let mut buzzer_pin = Output::new(peripherals.GPIO18, Level::Low, OutputConfig::default());
    let mut led = Output::new(peripherals.GPIO2, Level::Low, OutputConfig::default());

    loop {
        if sensor_pin.is_high() {
            buzzer_pin.set_high();
            led.set_high();
            blocking_delay(Duration::from_millis(100));
            buzzer_pin.set_low();
            led.set_low();
        }
        blocking_delay(Duration::from_millis(100));
    }
}

fn blocking_delay(duration: Duration) {
    let delay_start = Instant::now();
    while delay_start.elapsed() < duration {}
}
```

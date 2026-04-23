# 使用 ESP32 的电机控制脉宽调制器（MCPWM）外设控制舵机

在之前的练习中，我们使用 ESP32 的 LEDC 外设来控制舵机。在本练习中，我们将使用 MCPWM 来实现相同的功能。关于 MCPWM 的介绍、其操作以及 esp-hal 中对应的函数，请参阅[这里](../core-concepts/pwm/mcpwm.md)。请在继续之前阅读该章节。


### 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 servo-mcpwm
```

这将打开一个屏幕，要求你选择选项。在最新的 esp-hal 中，mcpwm 需要我们显式启用不稳定特性（unstable features）。

- 因此请选择 "Enable unstable HAL features."

然后按键盘上的 "s" 保存。

### 时钟配置

让我们创建一个外设时钟实例。我们选择 1 MHz 作为 PWM 的基础时钟频率。该函数将在内部计算预分频器（Prescaler）并分割输入时钟，即 160 MHz。

```rust
let clock_cfg = PeripheralClockConfig::with_frequency(Rate::from_mhz(32)).unwrap();
```

对于舵机，我们需要实现最终 50 Hz 的 PWM 频率。因此，我们需要将基础时钟频率尽可能降低。使用最大预分频器值 255，我们可以降到 625 kHz，但我们将其保持在 1 MHz 以使计算更简单。

### 配置 MCPWM 和引脚

我们将使用 MCPWM0 外设，选择 timer0 和 operator0。接下来，我们将配置它使用 GPIO33，并将 PWM 信号设置为高电平，直到达到我们在 PWM 周期中指定的时间戳值。

```rust
let mut mcpwm = McPwm::new(peripherals.MCPWM0, clock_cfg);
// 将 operator0 连接到 timer0
// connect operator0 to timer0
mcpwm.operator0.set_timer(&mcpwm.timer0);
// 将 operator0 连接到引脚
// connect operator0 to pin
let mut pwm_pin = mcpwm
    .operator0
    .with_pin_a(peripherals.GPIO33, PwmPinConfig::UP_ACTIVE_HIGH);
```

### 配置定时器

要为舵机实现 50 Hz 的 PWM 信号，使用 1 MHz 时钟，定时器总共需要计数 20,000 个时钟周期。由于定时器从 0 计数到 19,999，周期设置为 19_999，总共为 20,000 个时钟周期。

```rust
let timer_clock_cfg = clock_cfg
    .timer_clock_with_frequency(19_999, PwmWorkingMode::Increase, Rate::from_hz(50))
    .unwrap();
mcpwm.timer0.start(timer_clock_cfg);
```

### 舵机舵盘的旋转

要旋转舵机舵盘，我们调整 PWM 信号的时间戳。时间戳值对应所需的角度：

- 对于 0 度，我们将时间戳设置为 500（20,000 的 2.5%）。
- 对于 90 度，我们将时间戳设置为 1500（20,000 的 7.5%）。
- 对于 180 度，我们将时间戳设置为 2500（20,000 的 12.5%）。

每次调整后，我们给予足够的延时，让舵机到达指定位置。

```rust
loop {
    // 0 度（20_000 的 2.5% => 500）
    // 0 degree (2.5% of 20_000 => 500)
    pwm_pin.set_timestamp(500);
    delay.delay_millis(1500);

    // 90 度（20_000 的 7.5% => 1500）
    // 90 degree (7.5% of 20_000 => 1500)
    pwm_pin.set_timestamp(1500);
    delay.delay_millis(1500);

    // 180 度（20_000 的 12.5% => 2500）
    // 180 degree (12.5% of 20_000 => 2500)
    pwm_pin.set_timestamp(2500);
    delay.delay_millis(1500);
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

// MCPWM
use esp_hal::mcpwm::operator::PwmPinConfig;
use esp_hal::mcpwm::timer::PwmWorkingMode;
use esp_hal::mcpwm::{McPwm, PeripheralClockConfig};
use esp_hal::time::Rate;

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

    let delay = Delay::new();

    let clock_cfg = PeripheralClockConfig::with_frequency(Rate::from_mhz(32)).unwrap();
    let mut mcpwm = McPwm::new(peripherals.MCPWM0, clock_cfg);

    // 将 operator0 连接到 timer0
    // connect operator0 to timer0
    mcpwm.operator0.set_timer(&mcpwm.timer0);
    // 将 operator0 连接到引脚
    // connect operator0 to pin
    let mut pwm_pin = mcpwm
        .operator0
        .with_pin_a(peripherals.GPIO33, PwmPinConfig::UP_ACTIVE_HIGH);

    // 启动定时器，时间戳值范围为 0..=19999，频率为 50 Hz
    // start timer with timestamp values in the range of 0..=19999 and a frequency
    // of 50 Hz
    let timer_clock_cfg = clock_cfg
        .timer_clock_with_frequency(19_999, PwmWorkingMode::Increase, Rate::from_hz(50))
        .unwrap();
    mcpwm.timer0.start(timer_clock_cfg);

    loop {
        // 0 度（20_000 的 2.5% => 500）
        // 0 degree (2.5% of 20_000 => 500)
        pwm_pin.set_timestamp(500);
        delay.delay_millis(1500);

        // 90 度（20_000 的 7.5% => 1500）
        // 90 degree (7.5% of 20_000 => 1500)
        pwm_pin.set_timestamp(1500);
        delay.delay_millis(1500);

        // 180 度（20_000 的 12.5% => 2500）
        // 180 degree (12.5% of 20_000 => 2500)
        pwm_pin.set_timestamp(2500);
        delay.delay_millis(1500);
    }
}
```

## 克隆现有项目

你也可以克隆（或参考）我创建的项目，并导航到 `servo-mcpwm` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/servo-mcpwm
```

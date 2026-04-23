## 编写 Rust 代码在 ESP32 上实现 LED 渐变效果

现在到了有趣的部分；让我们开始编码吧！

### 使用 esp-generate 生成项目

你已经在快速入门部分完成了这一步。

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 led-fader
```

这将打开一个屏幕，要求你选择选项。在最新的 esp-hal 中，ledc 需要我们显式启用不稳定特性（unstable features）。

- 因此请选择"启用不稳定 HAL 特性（Enable unstable HAL features）"选项。

然后按键盘上的 "s" 保存。

> [!Tip]
> 在解释中，我不会包含所需的导入，因为它们不是特别有趣或需要太多解释。你始终可以参考下面的"完整代码"部分，或克隆项目来进行交叉检查。


## 自动生成的代码

当你使用 esp-generate 生成项目时，它会设置基本结构、配置 CPU 时钟，并包含外设的样板代码（boilerplate code），这样你就不必每次都手动输入这些内容。在本书中，我们将始终使用 esp-generate 创建项目，并在此基础上进行构建。

```rust
let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
let peripherals = esp_hal::init(config);
```

## LED 引脚

接下来，我们从外设实例中获取所需的 GPIO。在本例中，我们要点亮开发板的板载 LED（onboard LED），它连接到 GPIO 2。

```rust
let led = peripherals.GPIO2;
```

### PWM 配置

在本练习中，我们将使用低速 PWM 通道。首先，我们需要设置时钟源。esp-hal 库为低速时钟源定义了 `LSGlobalClkSource` 枚举，目前它只有一个值：`APBClk`。

```rust
ledc.set_global_slow_clock(LSGlobalClkSource::APBClk);
```

接下来，我们配置定时器。由于我们使用的是低速 PWM 通道，显然需要使用低速定时器。我们还需要指定使用哪个低速定时器（从 0 到 3）。

```rust
let mut lstimer0 = ledc.timer::<LowSpeed>(timer::Number::Timer0);
```

在使用定时器之前，我们还需要进行一些配置。我们将频率设置为 24 kHz。对于这个频率，使用 APB 时钟时，公式给出的最大分辨率为 12 位，最小分辨率为 2 位。在 esp-hal 中，该频率使用 5 位 PWM 分辨率，我们将采用相同的设置。

```rust
lstimer0
    .configure(timer::config::Config {
        duty: timer::config::Duty::Duty5Bit,
        clock_source: timer::LSClockSource::APBClk,
        frequency: Rate::from_khz(24),
    })
    .unwrap();
```

### PWM 通道

接下来，我们配置 PWM 通道。我们将使用 channel0，并用选定的定时器和初始占空比百分比"10%"进行设置。此外，我们将引脚配置设置为推挽输出（PushPull）。

```rust
let mut channel0 = ledc.channel(channel::Number::Channel0, led);
channel0
    .configure(channel::config::Config {
        timer: &lstimer0,
        duty_pct: 10,
        drive_mode: DriveMode::PushPull,
    })
    .unwrap();
```

### 渐变（Fading）

esp-hal 提供了一个名为 `start_duty_fade` 的函数，这让我们的工作更加轻松。否则，我们就必须手动在循环中以固定间隔递增和递减占空比。该函数逐渐从一个占空比百分比变化到另一个。它还接受第三个参数，指定从一个占空比过渡到另一个所需的时间。

```rust
channel0.start_duty_fade(0, 100, 1000).unwrap();
```

我们将在循环中运行此操作，并使用 HAL 提供的另一个函数 `is_duty_fade_running`；它返回一个布尔值，表示占空比渐变是否完成。

```rust
while channel0.is_duty_fade_running() {}
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
use esp_hal::ledc::{LSGlobalClkSource, Ledc, LowSpeed, channel, timer};

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

    let led = peripherals.GPIO2;
    // let led = peripherals.GPIO5;

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

    loop {
        channel0.start_duty_fade(0, 100, 1000).unwrap();
        while channel0.is_duty_fade_running() {}
        channel0.start_duty_fade(100, 0, 1000).unwrap();
        while channel0.is_duty_fade_running() {}
    }
}
```


## 克隆现有项目
你也可以克隆（或参考）我创建的项目，并导航到 `led-fader` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/led-fader
```

## 烧录
将代码烧录（flash）到 ESP32 后，你应该能看到板载 LED 上的渐变效果。

```rust
cargo run --release
```

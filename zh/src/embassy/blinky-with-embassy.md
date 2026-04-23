# 使用 ESP RTOS（Embassy）在 ESP32 上用 Rust 闪烁 LED

让我们用 embassy 支持重新设置闪烁 LED 项目。

### 设置项目

要启动项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 blinky-embassy
```

这次，我们需要选择 "Add Embassy framework support"。由于这需要不稳定特性，我们将首先点击 "Enable unstable HAL features"，然后继续选择 Embassy 支持。最后，我们按 's' 保存生成的带有 Embassy 支持的项目。

## 关键点

如果你注意到，main 函数现在标记为 async，代码中还有一些其他更改。然而，闪烁 LED 的核心逻辑保持不变。

关键新增部分是：

```rust
let timg0 = TimerGroup::new(peripherals.TIMG0);
esp_rtos::start(timg0.timer0);
```

这设置了一个 Embassy 处理异步任务（如延时）所需的定时器。我们使用硬件定时器 TIMG0 创建一个定时器组，然后将其一个定时器传递给 esp_rtos::start，让 Embassy 将其用于基于时间的操作。


## 完整代码

```rust
#![no_std]
#![no_main]
#![deny(
    clippy::mem_forget,
    reason = "mem::forget is generally not safe to do with esp_hal types, especially those \
    holding buffers for the duration of a data transfer."
)]

use embassy_executor::Spawner;
use embassy_time::{Duration, Timer};
use esp_hal::clock::CpuClock;
use esp_hal::gpio::{Level, Output, OutputConfig};
use esp_hal::timer::timg::TimerGroup;

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
async fn main(_spawner: Spawner) -> ! {
    // generator version: 1.0.0

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    let timg0 = TimerGroup::new(peripherals.TIMG0);
    esp_rtos::start(timg0.timer0);

    let mut led = Output::new(peripherals.GPIO2, Level::High, OutputConfig::default());

    loop {
        led.toggle();
        Timer::after(Duration::from_secs(1)).await;
    }
}
```


## 克隆现有项目

你可以克隆（或参考）我创建的项目并导航到 `async-projects/blinky-embassy` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/async-projects/blinky-embassy
```

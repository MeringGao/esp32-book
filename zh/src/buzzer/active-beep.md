## 使用 ESP32 和 Rust 让有源蜂鸣器发出蜂鸣声

既然你已经知道有源蜂鸣器（active buzzer）使用简单，只需通电就能让它蜂鸣。在本练习中，我们将用少量代码让它发出蜂鸣声。


### 硬件需求
- **有源蜂鸣器（Active Buzzer）**
- **母对公**或**公对公**跳线（jumper wires）（取决于你的设置）

### 使用 esp-generate 生成项目

你已经在快速入门部分完成了这一步。

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 active-buzzer
```

这将打开一个屏幕，要求你选择选项。目前，我们不需要选择任何选项。只需按键盘上的 "s" 保存即可。

## 代码

我们将 GPIO 33 设置为输出引脚，初始状态为低电平。这是我们连接蜂鸣器正极的引脚。

```rust
let mut buzzer = Output::new(peripherals.GPIO33, Level::Low, OutputConfig::default());
```

逻辑很简单：将蜂鸣器引脚设置为高电平 500 毫秒，然后再设置为低电平 500 毫秒，循环进行。这会使蜂鸣器发出蜂鸣声。

```rust
   loop {
        buzzer.set_high();
        blocking_delay(Duration::from_millis(500));
        buzzer.set_low();
        blocking_delay(Duration::from_millis(500));
    }
```

## 延时辅助函数
它等待直到指定的持续时间过去。

```rust
fn blocking_delay(duration: Duration) {
    let delay_start = Instant::now();
    while delay_start.elapsed() < duration {}
}
```

## 克隆现有项目
你可以克隆（或参考）我创建的项目，并导航到 `active-buzzer` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/active-buzzer
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
use esp_hal::gpio::{Level, Output, OutputConfig};
use esp_hal::main;
use esp_hal::time::{Duration, Instant};

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

    let mut buzzer = Output::new(peripherals.GPIO33, Level::Low, OutputConfig::default());

    loop {
        buzzer.set_high();
        blocking_delay(Duration::from_millis(500));
        buzzer.set_low();
        blocking_delay(Duration::from_millis(500));
    }
}

fn blocking_delay(duration: Duration) {
    let delay_start = Instant::now();
    while delay_start.elapsed() < duration {}
}
```

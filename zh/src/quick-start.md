# 快速入门 - Hello Embedded World！

在深入了解一切如何工作的理论和概念之前，让我们直接开始行动。使用这段简单的代码来点亮 ESP32 DevKit 的板载 LED（Onboard LED）。

### 设置项目

要启动项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 esp32-quick
```

这将打开一个屏幕，要求你选择选项。目前，我们不需要选择任何选项。只需在键盘上按 "s" 保存即可。

> 你也可以选择编辑器集成选项，然后选择你正在使用的特定编辑器。

接下来，导航到项目文件夹：
```sh
cd esp32-quick
```

## 完整代码

打开 `src/bin/main.rs` 文件。你会在里面找到一个简单的 "Hello, World" 程序。我们将用一个不同的嵌入式世界 "Hello, World" 来替换它——让板上的 LED 闪烁。只需将下面的代码复制并粘贴到 main.rs 文件中。

现在不用担心代码。我们将在下一章解释一切。目前，我们只是想看到一些令人兴奋的事情发生！

```rust
#![no_std]
#![no_main]
#![deny(
    clippy::mem_forget,
    // 中文注释
    // Original English comment: mem::forget is generally not safe to do with esp_hal types, especially those \
    // holding buffers for the duration of a data transfer.
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

// 创建一个默认的应用描述符（App Descriptor），这是 ESP-IDF 引导加载程序（Bootloader）所必需的。
// This creates a default app-descriptor required by the esp-idf bootloader.
// 更多信息请参阅：<https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
// For more information see: <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
esp_bootloader_esp_idf::esp_app_desc!();

#[main]
fn main() -> ! {

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    let mut led = Output::new(peripherals.GPIO2, Level::High, OutputConfig::default());

    loop {
        led.toggle();

        blocking_delay(Duration::from_millis(500));
    }

}

fn blocking_delay(duration: Duration) {
    let delay_start = Instant::now();
    while delay_start.elapsed() < duration {}
}

```

## 克隆现有项目

如果你遇到任何问题，你可以克隆（或参考）我创建的项目。

```sh
git clone https://github.com/ImplFerris/esp32-quick
cd esp32-quick/
```

## 烧录（Flash）- `Run Rust Run`

剩下的就是将代码烧录（Flash）到我们的设备上，然后看着它运行！板载 LED（Onboard LED）应该开始闪烁。

从你的项目文件夹运行以下命令：
```rust
cargo run
```

要在发布模式（Release Mode）下运行：
```rust
cargo run --release
```

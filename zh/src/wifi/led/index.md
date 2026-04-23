# 通过 Wi-Fi 控制 ESP32 LED

您可以在 ESP32 上配置一个 Web 服务器，通过 Web 请求接收指令，从而控制连接的设备或执行特定操作。例如，您可以使用电脑或手机上的浏览器发送命令来打开或关闭 LED、调整电机速度或获取传感器数据。

在本节中，我们将创建一个简单的网页，允许我们打开或关闭 LED。

## 项目基础

这次，我们不会使用 esp-generate 来设置项目。相反，我们将复制 webserver-base 项目并在此基础上进行开发。

我建议您在继续之前阅读以下部分；这将避免不必要的代码和解释重复。
- [创建 Web 服务器](../web-server/index.md)
- [分配静态 IP](../static-ip.md)


```sh
git clone https://github.com/ImplFerris/esp32-projects
cp -r esp32-projects/webserver-base ~/YOUR_PROJECT_FOLDER/wifi-led
```

**项目结构**：

```
├── build.rs
├── Cargo.toml
├── rust-toolchain.toml
├── src
│   ├── bin
│   │   └── main.rs
│   ├── index.html
│   ├── led.rs
│   ├── lib.rs
│   ├── web.rs
│   └── wifi.rs
```

## Serde
Serde 是一个用于序列化（serializing）和反序列化（deserializing）数据结构的 Rust crate。我们将使用它来处理后端和前端之间交换的 JSON 数据。

更新 Cargo.toml：
```toml
serde = { version = "1.0.228", default-features = false, features = ["derive"] }

# 启用 "json" 功能
# Enable The "json" feature
picoserve = { version = "0.17.1", features = ["embassy", "json"] }
```

## LED 任务

我将这些代码放在 "led.rs" 模块中。

首先，我们将创建一个 Embassy 任务，根据存储在 LED_STATE 变量中的值来切换板载 LED（onboard LED）的状态，该变量将使用 AtomicBool 类型。"原子类型（Atomic types）提供线程之间的原始共享内存通信，是其他并发类型的构建块"。要了解更多关于原子类型的信息，请参阅 Rust 标准库文档中关于 [atomics](https://doc.rust-lang.org/beta/core/sync/atomic/index.html) 的内容或 [Rust Atomics and Locks](https://marabos.nl/atomics/) 一书。

```rust
use core::sync::atomic::{AtomicBool, Ordering};

use embassy_time::{Duration, Timer};
use esp_hal::gpio::Output;

pub static LED_STATE: AtomicBool = AtomicBool::new(false);

#[embassy_executor::task]
pub async fn led_task(mut led: Output<'static>) {
    loop {
        if LED_STATE.load(Ordering::Relaxed) {
            led.set_high();
        } else {
            led.set_low();
        }
        Timer::after(Duration::from_millis(50)).await;
    }
}
```

在 led_task 函数中，我们接收一个 LED 引脚作为参数，并在循环中持续检查 LED_STATE 变量的值。我们使用 Ordering::Relaxed 的 load 方法读取该值。如果值为 true，我们打开 LED；否则，关闭 LED。

在 main 函数中，我们 spawn（生成）led_task 以在后台运行它。我们将传递 GPIO 2 引脚（如果您想使用外接 LED（external LED），请将其替换为您连接 LED 的引脚），即板载 LED，并将 LED 的初始状态设置为低电平（Low）。

```rust
// LED 任务
let led = Output::new(peripherals.GPIO2, Level::Low, OutputConfig::default());
spawner.must_spawn(lib::led::led_task(led));
```















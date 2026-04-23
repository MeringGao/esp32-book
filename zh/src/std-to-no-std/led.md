# LED

下一步是配置 DevKit v1 的板载 LED（onboard LED）。我们要怎么做呢？为此，我们需要控制一个 GPIO 引脚。

## GPIO
GPIO 代表 General Purpose Input/Output（通用输入/输出）。这些是微控制器上的引脚，可以读取数字信号（输入）或发送数字信号（输出）。

- 想读取一个按钮？使用 GPIO 作为输入。

- 想点亮一个 LED？使用 GPIO 作为输出。

- 想与传感器通信或控制电机？GPIO 引脚就是起点。

GPIO 是你的程序与外部世界对话的最基本也是最重要的方式。

## 板载 LED 连接到哪个引脚？

如果你使用的是裸 ESP32 芯片，没有板载 LED。

但是，我们使用的是 ESP32 DevKit v1 开发板。在这个开发板上，有一个板载 LED（onboard LED）连接到 GPIO2。所以这就是我们接下来要使用的引脚。

## 初始化 LED

为了控制板载 LED，我们将 GPIO2 配置为输出引脚。

添加这个必要的导入：

```rust
use esp_hal::gpio::{Level, Output, OutputConfig};
use esp_hal::time::{Duration, Instant};
```

在 main 函数中添加以下行：

```rust
let mut led = Output::new(peripherals.GPIO2, Level::High, OutputConfig::default());
```

`peripherals.GPIO2` 部分让我们可以访问 GPIO2 引脚。我们将其初始状态设置为 `Level::High`，这会点亮 LED。`OutputConfig::default()` 使用标准设置将其配置为输出引脚。请注意，我们将 led 变量标记为可变的（mutable），因为我们稍后会更改它的状态。我们可以使用 `toggle()` 函数来切换 LED 的高低状态，从而实现 LED 闪烁。

## 主循环

主循环使用一种简单的方法持续切换 LED 状态。我们在 LED 输出引脚上调用 `toggle()` 方法，它在高电平和低电平状态之间切换。每次切换之间，我们引入 500 毫秒的延时以产生可见的闪烁效果。

```rust
loop {
    led.toggle();
    blocking_delay(Duration::from_millis(500));
}
```

`blocking_delay` 函数通过忙等待（busy-waiting）直到指定的持续时间过去，从而提供了一种简单的定时机制。它捕获当前时间并持续检查时间是否已过：

```rust
fn blocking_delay(duration: Duration) {
    let delay_start = Instant::now();
    while delay_start.elapsed() < duration {}
}
```

## 最终代码

这是完整的代码：

```rust
#![no_std]
#![no_main]

use esp_hal::clock::CpuClock;
use esp_hal::gpio::{Level, Output, OutputConfig};
use esp_hal::main;
use esp_hal::time::{Duration, Instant};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

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

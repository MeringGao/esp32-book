## 打印摇杆移动的 ADC 值

在这个程序中，我们将实时观察摇杆移动如何影响 ADC 值。我们将把 ESP32 与摇杆连接起来。

当你移动摇杆时，对应的 ADC 值将打印在系统控制台中。你可以将这些值与[之前的移动与 ADC 示意图](./movement-and-12-bit-adc-value.md)进行比较；它们应该与图中显示的值大致匹配。按下摇杆帽将打印 **"Button Pressed"** 以及当前坐标。

## 使用 esp-generate 生成项目

我们将为此项目启用 async（Embassy）支持。要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 joystick-movement
```

这将打开一个屏幕，要求你选择选项。

- 选择 "Enable unstable HAL features"
- 然后选择 "Adds embassy framework support"。

按键盘上的 "s" 保存即可。

## 更新 cargo.toml

nb crate 通过返回 nb::Result 来简化非阻塞 I/O（例如读取传感器、UART 数据），当操作未就绪时返回 WouldBlock 错误，允许你稍后重试而不会阻塞。

如果你想知道为什么需要它，ADC 的 read_oneshot 函数返回一个 nb::Result，如果尚未就绪，可能会返回 nb::Error::WouldBlock。用 nb::block 包装它会使你的代码不断重试，直到 ADC 完成工作并返回正确的结果。

```toml
nb = "1.1.0"
```

### 配置 ADC

让我们设置 ADC 并配置映射到摇杆 VRX 和 VRY 引脚的 GPIO 13 和 GPIO 14：

```rust
let mut adc2_config = AdcConfig::new();
let mut vrx_pin = adc2_config.enable_pin(peripherals.GPIO13, Attenuation::_11dB);
let mut vry_pin = adc2_config.enable_pin(peripherals.GPIO14, Attenuation::_11dB);

let mut adc2 = Adc::new(peripherals.ADC2, adc2_config);

```

我们还将 GPIO15 配置为按钮的上拉输入（pull-up input）：

```rust
let btn = Input::new(
    peripherals.GPIO32,
    InputConfig::default().with_pull(Pull::Up),
);
```

### 打印坐标

我们希望在 vrx 或 vry 值变化超过一定阈值时才打印坐标。这样可以避免连续打印不必要的值。

为此，我们初始化变量来存储之前的值，并设置一个标志来确定何时打印：

```rust
let mut prev_vrx: u16 = 0;
let mut prev_vry: u16 = 0;
let mut prev_btn_state = false;
let mut print_vals = true;
```

**读取 ADC 值：**

首先，读取 vrx 和 vry 的 ADC 值。如果读取操作期间出现错误，我们忽略它并继续循环：

```rust
let Ok(vry): Result<u16, _> = nb::block!(adc2.read_oneshot(&mut vry_pin)) else {
    continue;
};
let Ok(vrx): Result<u16, _> = nb::block!(adc2.read_oneshot(&mut vrx_pin)) else {
    continue;
};
```

**检查阈值变化：**

接下来，我们检查 vrx 或 vry 的当前值与先前值之间的绝对差是否超过阈值（例如 100）。如果是，我们更新先前值并将 print_vals 标志设置为 true：

```rust
if vrx.abs_diff(prev_vrx) > 100 {
    prev_vrx = vrx;
    print_vals = true;
}

if vry.abs_diff(prev_vry) > 100 {
    prev_vry = vry;
    print_vals = true;
}
```

使用阈值可以过滤掉小的 ADC 波动，避免不必要的打印，并确保只在有显著变化时才更新。

**打印坐标**

如果 print_vals 为 true，我们将其重置为 false 并通过 USB 串口打印 X 和 Y 坐标：

```rust
if print_vals {
    print_vals = false;
    println!("X: {} Y: {}\r\n", vrx, vry);
}
```

### 使用状态转换检测按钮按下

按钮通常处于高电平状态。当你按下摇杆按钮时，它从高电平切换到低电平。然而，由于程序在循环中运行，简单地检查按钮是否为低电平可能导致多次检测到按下。为了避免这种情况，我们只通过检测从高到低的转换来注册一次按下，这表明按钮已被按下。

为此，我们跟踪按钮的先前状态，并在打印 "button pressed" 消息之前将其与当前状态进行比较。如果按钮当前处于低电平（按下）且先前状态为高电平（未按下），我们将其识别为新的按下并打印消息。然后，我们将先前状态更新为当前状态，确保正确检测未来的转换。

```rust
let btn_state = btn.is_low();
if btn_state && !prev_btn_state {
    println!("Button Pressed");
    print_vals = true;
}
prev_btn_state = btn_state;
```

## 克隆现有项目

你可以克隆（或参考）我创建的项目，并导航到 `joystick-movement` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/joystick-movement/
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

use defmt::info;
use embassy_executor::Spawner;
use embassy_time::{Duration, Timer};
use esp_hal::clock::CpuClock;
use esp_hal::timer::timg::TimerGroup;
use esp_println::{self as _, println};

use esp_hal::analog::adc::{Adc, AdcConfig, Attenuation};
use esp_hal::gpio::{Input, InputConfig, Pull};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

// 创建一个 esp-idf 引导加载程序所需的默认应用描述符。
// 更多信息请参阅：<https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
// This creates a default app-descriptor required by the esp-idf bootloader.
// For more information see: <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
esp_bootloader_esp_idf::esp_app_desc!();

#[esp_rtos::main]
async fn main(spawner: Spawner) -> ! {
    // generator version: 1.0.0

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    let timg0 = TimerGroup::new(peripherals.TIMG0);
    esp_rtos::start(timg0.timer0);

    info!("Embassy initialized!");

    let _ = spawner;

    let btn = Input::new(
        peripherals.GPIO32,
        InputConfig::default().with_pull(Pull::Up),
    );

    let mut adc2_config = AdcConfig::new();
    let mut vrx_pin = adc2_config.enable_pin(peripherals.GPIO13, Attenuation::_11dB);
    let mut vry_pin = adc2_config.enable_pin(peripherals.GPIO14, Attenuation::_11dB);

    let mut adc2 = Adc::new(peripherals.ADC2, adc2_config);

    let mut prev_vrx: u16 = 0;
    let mut prev_vry: u16 = 0;
    let mut prev_btn_state = false;
    let mut print_vals = true;

    loop {
        let Ok(vry): Result<u16, _> = nb::block!(adc2.read_oneshot(&mut vry_pin)) else {
            continue;
        };
        let Ok(vrx): Result<u16, _> = nb::block!(adc2.read_oneshot(&mut vrx_pin)) else {
            continue;
        };

        if vrx.abs_diff(prev_vrx) > 100 {
            prev_vrx = vrx;
            print_vals = true;
        }

        if vry.abs_diff(prev_vry) > 100 {
            prev_vry = vry;
            print_vals = true;
        }

        let btn_state = btn.is_low();
        if btn_state && !prev_btn_state {
            println!("Button Pressed");
            print_vals = true;
        }
        prev_btn_state = btn_state;

        if print_vals {
            print_vals = false;

            println!("X: {} Y: {}\r\n", vrx, vry);
        }

        Timer::after(Duration::from_millis(50)).await;
    }
}
```

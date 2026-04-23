# 在 OLED 上显示 "Hello, Rust!"

本练习作为 OLED 显示屏的简单入门，我们将通过在屏幕上显示 "Hello, Rust!" 来保持简洁。

## 使用 esp-generate 生成项目

我们将为这个项目启用异步（Embassy）支持。要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 hello-oled
```

这将打开一个选项选择界面。

- 选择 "Enable unstable HAL features" 选项
- 然后选择 "Adds embassy framework support" 选项

按键盘上的 "s" 保存即可。

按键盘上的 "s" 保存即可。

## 更新 Cargo.toml

```toml
ssd1306 = { version = "0.10.0", features = ["async"] }
embedded-graphics = "0.8.1"
```

## 导入模块

首先添加必要的导入：

```rust
// I2C
use esp_hal::i2c::master::Config as I2cConfig; // 为了方便，使用别名导入
use esp_hal::i2c::master::I2c;
use esp_hal::time::Rate;

// OLED
use ssd1306::{I2CDisplayInterface, Ssd1306Async, prelude::*};

// Embedded Graphics
use embedded_graphics::{
    mono_font::{MonoTextStyleBuilder, ascii::FONT_6X10},
    pixelcolor::BinaryColor,
    prelude::Point,
    prelude::*,
    text::{Baseline, Text},
};
```

## 初始化 I2C

我们初始化 I2C 接口以实现 ESP32 与 OLED 显示屏之间的通信。I2C 总线配置为 400 kHz 的频率，超时时间为 100 个总线时钟周期。我们将 GPIO18 分配给 SCL（Serial Clock Line，串行时钟线），GPIO23 分配给 SDA（Serial Data Line，串行数据线），并为接口启用异步操作。

```rust
let i2c_bus = I2c::new(
        peripherals.I2C0,
        // I2cConfig 是 esp_hal::i2c::master::I2c::Config 的别名
        I2cConfig::default().with_frequency(Rate::from_khz(400)),
    )
    .unwrap()
    .with_scl(peripherals.GPIO18)
    .with_sda(peripherals.GPIO23)
    .into_async();
```

## 初始化 ssd1306 驱动

接下来，我们将使用辅助结构体 `I2CDisplayInterface` 为显示屏创建一个预配置的 I2C 接口。然后，我们使用 `Ssd1306Async` 结构体（非异步版本请使用 `Ssd1306`），并传入我们创建的接口实例、显示屏尺寸 `DisplaySize128x64`，以及显示屏旋转方向。由于我们不需要旋转，将其设置为 `DisplayRotation::Rotate0`。

ssd1306 crate 支持三种显示模式：
- **BasicMode**：提供带有底层方法的基本控制
- **BufferedGraphicsMode**：使用帧缓冲区（framebuffer）进行高级绘制，并与 embedded-graphics 集成
- **TerminalMode**：一种无缓冲模式，设计用于像终端一样绘制文本和设置光标位置

在本练习中，我们将使用 BufferedGraphicsMode。

接下来，我们调用 `init()` 函数来初始化并清空图形模式下的显示屏。

```rust
let interface = I2CDisplayInterface::new(i2c_bus);
// 初始化显示屏
let mut display = Ssd1306Async::new(interface, DisplaySize128x64, DisplayRotation::Rotate0)
    .into_buffered_graphics_mode();
display.init().await.unwrap();
```

### 文本样式与位置

我们将使用等宽字体（monospaced font）来显示文本。`MonoTextStyleBuilder` 将帮助我们创建文本样式，这里使用 6x10 像素的字体大小。你可以在[这里](https://docs.rs/embedded-graphics/latest/embedded_graphics/mono_font/ascii/index.html)找到其他等宽字体。

如果你使用的是彩色 OLED 显示屏，可以指定不同的字体颜色。但由于我们使用的是单色（monochrome）显示屏，因此使用 `BinaryColor::On` 将文本颜色设置为白色。这实际上就是开启显示文本所需的那些像素。

```rust
let text_style = MonoTextStyleBuilder::new()
    .font(&FONT_6X10)
    .text_color(BinaryColor::On)
    .build();

Text::with_baseline("Hello, Rust!", Point::new(0, 16), text_style, Baseline::Top)
    .draw(&mut display)
    .unwrap();
```

基线（baseline）是一条假想的线，用于确定文本的对齐位置。我们设置了基线，x 位置为 0，y 位置为 16。我们还指定了文本在该空间内的对齐方式。`Baseline` 枚举控制文本在基线中的定位方式。例如，使用 `Baseline::Top` 会将文本的顶部与起始点对齐，而 `Baseline::Bottom` 会将文本的底部与起始点对齐。它还有其他选项，如 `Middle`、`Alphabetic`。

我建议你调整点（Point）的值和 `Baseline` 的值，观察它们对显示效果的影响。视觉上的变化会让你理解得更清晰。

接下来，我们可以在任何实现了 `DrawTarget` trait 的对象上绘制文本。ssd1306 的 `BufferedGraphicsMode` 实现了这个 trait，因此我们可以将 display 作为可变引用传递给 `draw` 函数。

## 刷新（Flush）

最后，我们调用 `flush` 函数，将数据写入显示屏。只有执行此操作后，更新后的内容才会显示在 OLED 屏幕上。

```rust
display.flush().await.unwrap();
```

## 克隆已有项目

你也可以克隆（或参考）我创建的项目，并导航到 `hello-oled` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/hello-oled
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

use defmt::info;
use embassy_executor::Spawner;
use embassy_time::{Duration, Timer};

use esp_hal::clock::CpuClock;
use esp_hal::timer::timg::TimerGroup;
use esp_println as _;

// I2C
use esp_hal::i2c::master::Config as I2cConfig; // 为了方便，使用别名导入
use esp_hal::i2c::master::I2c;
use esp_hal::time::Rate;

// OLED
use ssd1306::{I2CDisplayInterface, Ssd1306Async, prelude::*};

// Embedded Graphics
use embedded_graphics::{
    mono_font::{MonoTextStyleBuilder, ascii::FONT_6X10},
    pixelcolor::BinaryColor,
    prelude::Point,
    prelude::*,
    text::{Baseline, Text},
};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

// 创建 ESP-IDF 引导加载程序所需的应用描述符。
// 更多信息请参阅：<https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
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

    let i2c_bus = I2c::new(
        peripherals.I2C0,
        // I2cConfig 是 esp_hal::i2c::master::I2c::Config 的别名
        I2cConfig::default().with_frequency(Rate::from_khz(400)),
    )
    .unwrap()
    .with_scl(peripherals.GPIO18)
    .with_sda(peripherals.GPIO23)
    .into_async();

    let interface = I2CDisplayInterface::new(i2c_bus);
    // 初始化显示屏
    let mut display = Ssd1306Async::new(interface, DisplaySize128x64, DisplayRotation::Rotate0)
        .into_buffered_graphics_mode();
    display.init().await.unwrap();

    let text_style = MonoTextStyleBuilder::new()
        .font(&FONT_6X10)
        .text_color(BinaryColor::On)
        .build();

    Text::with_baseline("Hello, Rust!", Point::new(0, 16), text_style, Baseline::Top)
        .draw(&mut display)
        .unwrap();

    display.flush().await.unwrap();

    loop {
        Timer::after(Duration::from_secs(1)).await;
    }
}
```

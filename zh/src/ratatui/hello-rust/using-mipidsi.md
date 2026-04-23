# 使用 `mipidsi` crate

本教程展示如何将较新的 `mipidsi` crate 与 TFT 和 ESP32 一起使用。

`mipidsi` crate 是一个统一的 crate，支持多种显示屏，也支持 `ili9341`。

## 设置

设置与上一个程序相同，只是对 `Cargo.toml` 做了一些更改。

### `Cargo.toml` 依赖项

```toml
embedded-hal-bus = "0.3.0"
embedded-graphics = "0.8.1"
mipidsi = "0.9.0"
static_cell = "2.1.1"
```

`static_cell` crate 用于提供一种安全的方式来分配堆内存或进行动态内存管理。

这是必需的，因为 SPI 缓冲区需要在整个程序运行期间存在，但直到运行时设置好外设后才能初始化。


#### `mousefood` 和 `ratatui`

将以下 `mousefood` 和 `ratatui` 版本添加到 `Cargo.toml`

```toml
mousefood = { git = "https://github.com/j-g00da/mousefood", rev = "cc9f8fe372f09342537bc31a1355f77f2693d70b", default-features = false, features = [
  "fonts",
] }
ratatui = { version = "0.30.0-alpha.5", default-features = false }

```

## 导入

```rust
use static_cell::StaticCell;
use embedded_hal_bus::spi::ExclusiveDevice;

// ESP 相关
// ESP Stuff
use esp_hal::{
  delay::Delay,
  spi::{
    master::{
      Config as SpiConfig,
      Spi
    },
    Mode as SpiMode,
  },
  time::Rate,
  gpio::{
    Level,
    Output,
    OutputConfig
  },
  clock::CpuClock,
  main
};

// Embedded graphics 相关
// Embedded graphics stuff
use embedded_graphics::pixelcolor::Rgb565;
use embedded_graphics::prelude::*;

// TFT 屏幕相关
// TFT Screen stuff
use mipidsi::{Builder, models::ILI9342CRgb565, interface::SpiInterface, options::{Orientation, Rotation}};

// Mousefood 相关
// Mousefood stuff
use mousefood::{EmbeddedBackend, EmbeddedBackendConfig};
use ratatui::{layout::{Constraint, Flex, Layout}, widgets::{Block, Paragraph, Wrap}};
use ratatui::{style::*, Frame, Terminal};
```

这里有一些配置变量需要使用。这段代码在导入之后

```rust
static SPI_BUFFER: StaticCell<[u8; 512]> = StaticCell::new();
```

## 初始化 TFT 显示驱动

```rust
let spi = Spi::new(
  peripherals.SPI2,
  SpiConfig::default()
    .with_frequency(Rate::from_mhz(60))
    .with_mode(SpiMode::_0)
)
  .unwrap()
  .with_sck(peripherals.GPIO18)
  .with_mosi(peripherals.GPIO23);

let cs = Output::new(peripherals.GPIO5, Level::Low, OutputConfig::default());
let dc = Output::new(peripherals.GPIO2, Level::Low, OutputConfig::default());
let reset = Output::new(peripherals.GPIO4, Level::Low, OutputConfig::default());

let buffer = SPI_BUFFER.init([0; 512]);

let spi_dev = ExclusiveDevice::new_no_delay(spi, cs).unwrap();
let interface = SpiInterface::new(spi_dev, dc, buffer);

let mut display = Builder::new(
  ILI9342CRgb565,
  interface
)
  .reset_pin(reset)
  .init(&mut Delay::new())
  .unwrap();

// 关键：在清除和创建后端之前设置方向
// CRITICAL: Set orientation BEFORE clearing and creating backend
display.set_orientation(
  Orientation::default().rotate(Rotation::Deg270)
).unwrap();

// 用新方向清除
// Clear with the new orientation
display.clear(Rgb565::BLACK).unwrap();
```

> 这里使用 `ILI9342CRgb565` 驱动而不是 `ili9341` 驱动是由于兼容性问题。
> 如果出现显示问题，你可以将 `ILI9342CRgb565` 切换为 `ILI9341Rgb565`

## 创建后端

要创建后端，你可以使用 `mousefood` 嵌入式后端

```rust
let backend = EmbeddedBackend::new(&mut display, EmbeddedBackendConfig::default());
```

然后，你创建使用这个后端的 `ratatui` 终端

```rust
let mut terminal = Terminal::new(backend).unwrap();
```

## 绘制函数

这个函数绘制 TUI 的视觉元素。

```rust
fn draw(frame: &mut Frame) {
    let outer_block = Block::bordered()
        .title_style(Style::new().green())
        .title("ESP32 Dashboard");

    frame.render_widget(outer_block, frame.area());

    let vertical_layout = Layout::vertical([Constraint::Length(3)])
        .flex(Flex::Center)
        .split(frame.area());

    let horizontal_layout = Layout::horizontal([Constraint::Length(25)])
        .flex(Flex::Center)
        .split(vertical_layout[0]);

    let text = "Rat(a tui) inside ESP32";
    let paragraph = Paragraph::new(text.dark_gray())
        .wrap(Wrap { trim: true })
        .centered();

    let bordered_block = Block::bordered()
        .border_style(Style::new().yellow())
        .title("impl rust");

    frame.render_widget(paragraph.block(bordered_block), horizontal_layout[0]);

}
```

## 渲染

在 main 函数中，定义 `terminal` 之后

```rust
loop {
  terminal.draw(draw).unwrap();
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
use esp_hal::main;
use esp_println as _;

// Embedded Graphics 相关
// Embedded Graphics related
use embedded_graphics::pixelcolor::Rgb565;
use embedded_graphics::prelude::*;

// ESP32 SPI + 显示驱动桥接
// ESP32 SPI + Display Driver bridge
use embedded_hal_bus::spi::ExclusiveDevice;
use esp_hal::delay::Delay;
use esp_hal::spi::Mode as SpiMode;
use esp_hal::spi::master::Config as SpiConfig;
use esp_hal::spi::master::Spi;
use esp_hal::time::Rate; // 用于指定 SPI 频率

use static_cell::StaticCell;

// 用于 TFT 屏幕
// For TFT Screen
use mipidsi::{
    Builder,
    interface::SpiInterface,
    models::ILI9342CRgb565,
    options::{Orientation, Rotation},
};

// 用于管理 GPIO 状态
// For managing GPIO state
use esp_hal::gpio::{Level, Output, OutputConfig};

// 用于 ratatui
// For ratatui
use mousefood::{EmbeddedBackend, EmbeddedBackendConfig};
use ratatui::layout::{Constraint, Flex, Layout};
use ratatui::widgets::{Block, Paragraph, Wrap};
use ratatui::{Frame, Terminal, style::*};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

extern crate alloc;

// 这将创建一个 esp-idf 引导加载程序所需的默认应用描述符。
// 更多信息请参阅：<https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
// This creates a default app-descriptor required by the esp-idf bootloader.
// For more information see: <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
esp_bootloader_esp_idf::esp_app_desc!();

static SPI_BUFFER: StaticCell<[u8; 512]> = StaticCell::new();

#[main]
fn main() -> ! {
    // generator version: 1.0.0

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    esp_alloc::heap_allocator!(#[unsafe(link_section = ".dram2_uninit")] size: 98767);

    // 初始化 SPI
    // Initialize SPI
    let spi = Spi::new(
        peripherals.SPI2,
        SpiConfig::default()
            .with_frequency(Rate::from_mhz(60))
            .with_mode(SpiMode::_0),
    )
    .unwrap()
    //CLK
    .with_sck(peripherals.GPIO18)
    //DIN
    .with_mosi(peripherals.GPIO23);
    let cs = Output::new(peripherals.GPIO15, Level::Low, OutputConfig::default());
    let dc = Output::new(peripherals.GPIO2, Level::Low, OutputConfig::default());
    let reset = Output::new(peripherals.GPIO4, Level::Low, OutputConfig::default());

    let buffer = SPI_BUFFER.init([0; 512]);

    let spi_dev = ExclusiveDevice::new_no_delay(spi, cs).unwrap();
    let interface = SpiInterface::new(spi_dev, dc, buffer);

    let mut display = Builder::new(ILI9342CRgb565, interface)
        .reset_pin(reset)
        .init(&mut Delay::new())
        .unwrap();

    // 关键：在清除和创建后端之前设置方向
    // CRITICAL: Set orientation BEFORE clearing and creating backend
    display
        .set_orientation(Orientation::default().rotate(Rotation::Deg270))
        .unwrap();

    // 用新方向清除
    // Clear with the new orientation
    display.clear(Rgb565::BLACK).unwrap();

    let backend = EmbeddedBackend::new(&mut display, EmbeddedBackendConfig::default());

    let mut terminal = Terminal::new(backend).unwrap();
    loop {
        terminal.draw(draw).unwrap();
    }
}

fn draw(frame: &mut Frame) {
    let outer_block = Block::bordered()
        .border_style(Style::new().green())
        .title(" ESP32 Dashboard ");
    frame.render_widget(outer_block, frame.area());

    let vertical_layout = Layout::vertical([Constraint::Length(3)])
        .flex(Flex::Center)
        .split(frame.area());

    let horizontal_layout = Layout::horizontal([Constraint::Length(25)])
        .flex(Flex::Center)
        .split(vertical_layout[0]);

    let text = "Rat(a tui) inside ESP32";
    let paragraph = Paragraph::new(text.dark_gray())
        .wrap(Wrap { trim: true })
        .centered();

    let bordered_block = Block::bordered()
        .border_style(Style::new().yellow())
        .title(" impl Rust ");

    frame.render_widget(paragraph.block(bordered_block), horizontal_layout[0]);
}
```

## 克隆现有项目

你可以克隆（或参考）我创建的项目并导航到 `hello-rat-2` 文件夹。

```bash
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/hello-rat-2/
```

## 烧录程序

从你的项目文件夹（包含 `Cargo.toml`）运行以下命令以将程序烧录到 ESP32

```bash
cargo run --release
```


<img style="display: block; margin: auto;" src="../images/ratatui-on-esp32-embedded-rust-2.jpg" alt="Ratatui on ESP32"/>

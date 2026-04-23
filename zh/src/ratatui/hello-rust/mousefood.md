# Hello, Rat

在本章中，我们将编写一个简单的程序来开始使用 Ratatui。我们在这里只展示基础知识，看看它是如何工作的。

## 先决条件

- 你需要一块 TFT 显示屏来完成本章。如果你还没有完成 [TFT 显示屏章节](../../tft-display/index.md)，我建议先完成那个，然后再回到这里。由于 TFT 显示屏和 ESP32 之间的电路连接已经在那里解释过了，我们不会在这里重复这些说明。

- Ratatui 有一个不错的[入门教程](https://ratatui.rs/tutorials/hello-ratatui/)，用于构建 TUI 应用程序。如果你已经了解 Ratatui，这个练习非常简单——只是把东西拼在一起。如果你是新手，稍后请查看官方 Ratatui 教程以熟悉基础知识并构建更好的 UI。


## 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 hello-rat
```

这将打开一个屏幕，要求你选择选项。Ratatui 需要堆分配（heap allocation），因此我们将启用 "unstable" 和 "alloc" 特性。

- 首先，选择 "Enable unstable HAL features."
- 选择 "Enable allocations via the esp-alloc crate."

你也可以选择启用日志特性

- 所以，滚动到 "Flashing, logging and debugging (espflash)" 并按回车。
- 然后，选择 "Use defmt to print messages"。

只需按键盘上的 "s" 保存即可。

## 依赖项

首先，让我们用控制 TFT 显示屏所需的依赖项更新 Cargo.toml 文件。我们已经在前面的 TFT 显示屏部分介绍过这些了。

```toml
embedded-hal-bus = { version = "0.3" }
display-interface-spi = "0.5"
ili9341 = "0.6.0"
embedded-graphics = "0.8.1"
```

> [!Tip]
> 虽然这个例子使用 `ili9341` crate 与 TFT 显示屏交互，但你可以将 `mousefood` 与任何实现了 embedded-graphics trait 的显示驱动 crate 一起使用。

现在，让我们添加 ratatui 和 mousefood crate，它们让我们在嵌入式环境中使用 ratatui。

```toml
mousefood = { git = "https://github.com/j-g00da/mousefood", rev = "cc9f8fe372f09342537bc31a1355f77f2693d70b", default-features = false, features = [
  "fonts",
] }
ratatui = { version = "0.30.0-alpha.5", default-features = false }
```

对于 no_std 支持，你目前需要来自 GitHub 的最新 mousefood 代码和一个兼容的 ratatui alpha 版本。

## 导入

让我们引入设置和控制 TFT 显示屏所需的所有 crate 和模块。我们将导入 `embedded-graphics` 用于图形渲染，通过 `esp-hal` 进行 SPI 通信，以及 `ili9341` 显示驱动。最后，我们将包含 ratatui 相关的导入。

```rust
// Embedded Graphics 相关
// Embedded Graphics related
use embedded_graphics::pixelcolor::Rgb565;
use embedded_graphics::prelude::*;

// ESP32 SPI + 显示驱动桥接
// ESP32 SPI + Display Driver bridge
use esp_hal::delay::Delay;
use esp_hal::spi::master::Config as SpiConfig;
use esp_hal::spi::master::Spi;
use esp_hal::spi::Mode as SpiMode;
use esp_hal::time::Rate; // 用于指定 SPI 频率
use display_interface_spi::SPIInterface;
use embedded_hal_bus::spi::ExclusiveDevice;
use ili9341::{DisplaySize240x320, Ili9341, Orientation};

// 用于管理 GPIO 状态
// For managing GPIO state
use esp_hal::gpio::{Level, Output, OutputConfig};

// 用于 ratatui
// For ratatui
use mousefood::{EmbeddedBackend, EmbeddedBackendConfig};
use ratatui::layout::{Constraint, Flex, Layout};
use ratatui::widgets::{Block, Paragraph, Wrap};
use ratatui::{style::*, Frame, Terminal};
```

## 初始化 TFT 显示驱动

让我们通过首先设置 SPI 接口，然后创建和配置显示实例来初始化 TFT 显示屏。

```rust
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

  let spi_dev = ExclusiveDevice::new_no_delay(spi, cs).unwrap();
  let interface = SPIInterface::new(spi_dev, dc);

  let mut display = Ili9341::new(
      interface,
      reset,
      &mut Delay::new(),
      Orientation::Landscape,
      DisplaySize240x320,
  )
  .unwrap();
  display.clear(Rgb565::BLACK).unwrap();
```

## Ratatui 中的后端（Backends）

Ratatui 不直接与你的屏幕通信。相反，它使用一个称为"后端"（backend）的中介来处理所有低级别的终端操作。把后端想象成 Ratatui 高级绘制命令与实际终端或显示硬件之间的翻译器。

Ratatui 支持不同的后端，这使它足够灵活以在各种环境中工作。默认情况下，它使用 Crossterm 作为传统终端模拟器的后端。然而，这不适用于嵌入式系统。我们需要一个在嵌入式环境中工作的后端。这就是 mousefood crate 的用武之地，它为我们提供了嵌入式后端（Embedded Backend）。

这个后端允许 Ratatui 渲染到微控制器上的 LCD 屏幕、电子纸显示屏、小型 OLED 屏幕以及任何其他支持 embedded-graphics 库的显示硬件。

<img style="display: block; margin: auto;" src="../images/ratatui-embedded-backend.svg" alt="Ratatui Embedded Backend"/>

你可以在[这里](https://ratatui.rs/concepts/backends/)找到有关 Ratatui 后端的更多详情。

要让 Ratatui 使用 mousefood 嵌入式后端，你首先用你的显示屏的可变引用和默认配置初始化 EmbeddedBackend，像这样：

```rust
let backend = EmbeddedBackend::new(&mut display, EmbeddedBackendConfig::default());
```

然后，你用这个后端创建 Ratatui Terminal：

```rust
let mut terminal = Terminal::new(backend).unwrap();
```

## 绘制函数（Draw Function）

我们将定义 draw 函数，它设置我们 UI 帧的布局和视觉元素。我们首先创建一个外部绿色边框块，标题为 "ESP32 Dashboard"，包围整个界面。在这个块内，我们首先垂直然后水平组织布局以构建内容区域。

然后我们创建一个段落控件，显示文字 "Rat(a tui) inside ESP32,"。我们将这个段落包装在一个黄色边框块中，标题为 " impl Rust "，并在我们用布局定义的中心位置渲染它。

```rust
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

## 渲染（Rendering）

我们最后调用 Ratatui 的 draw 方法，传递我们的 draw 函数以将 UI 渲染到显示屏上。

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
use display_interface_spi::SPIInterface;
use embedded_hal_bus::spi::ExclusiveDevice;
use esp_hal::delay::Delay;
use esp_hal::spi::Mode as SpiMode;
use esp_hal::spi::master::Config as SpiConfig;
use esp_hal::spi::master::Spi;
use esp_hal::time::Rate; // 用于指定 SPI 频率
use ili9341::{DisplaySize240x320, Ili9341, Orientation};

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

    let spi_dev = ExclusiveDevice::new_no_delay(spi, cs).unwrap();
    let interface = SPIInterface::new(spi_dev, dc);

    let mut display = Ili9341::new(
        interface,
        reset,
        &mut Delay::new(),
        Orientation::Landscape,
        DisplaySize240x320,
    )
    .unwrap();
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

你可以克隆（或参考）我创建的项目并导航到 `hello-rat` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/hello-rat/
```

## 烧录程序

从你的项目文件夹运行以下命令以构建并烧录程序到你的 ESP32：

```rust
cargo run --release
```

你现在应该会在显示屏上看到 Ratatui 界面。

<img style="display: block; margin: auto;" src="../images/ratatui-on-esp32-embedded-rust.jpg" alt="Ratatui on ESP32"/>


如果你想使用 `mipidsi` crate 而不是 `ili9341` crate，请查看下一个[教程](./using-mipidsi.md)

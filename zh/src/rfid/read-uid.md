# 读取 UID

好了，让我们进入有趣的部分，开始动手实践！我们将先编写一个简单的程序来读取 RFID 标签的 UID。

## 使用 esp-generate 生成项目

我们将为此项目启用 async（Embassy）支持。要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 rfid-uid
```

这将打开一个屏幕，要求你选择选项。

- 选择 "Enable unstable HAL features"
- 然后选择 "Adds embassy framework support"。

按键盘上的 "s" 保存即可。

## 需要添加的额外 crate

更新你的 Cargo.toml，在现有依赖之外添加以下 crate：

```rust
mfrc522 = "0.8.0"
embedded-hal-bus = "0.2.0"
```

### mfrc522 驱动

我们将使用很棒的 crate "[mfrc522](https://crates.io/crates/mfrc522)"。它仍在开发中，但对于我们的需求来说已经足够了。

### embedded-hal-bus

要理解为什么需要 "embedded-hal-bus" crate，我们首先需要了解 Embedded HAL（Hardware Abstraction Layer，硬件抽象层）。Embedded HAL 提供了若干 trait，为控制微控制器上的常见外设（如 GPIO、PWM 和通信接口（如 I2C、SPI 和 UART））提供标准方式。这使得驱动程序可以在多种微控制器（如 ESP32、Raspberry Pi Pico）上兼容。

当我们想通过 SPI 与 RFID 标签通信时，embedded-hal 提供了 `SpiBus` 和 `SpiDevice` trait 来支持总线共享。SpiBus 代表整个总线，而 SpiDevice 代表总线上的一个设备。微控制器特定的 HAL（如 esp-hal）通常实现 SpiBus trait，而设备驱动（如 mfrc522）则实现 SpiDevice trait。

因此，我们需要从 SpiBus 获取 SpiDevice 才能用于 SD 卡。这就是 embedded-hal-bus crate 的用武之地。它提供了 SpiDevice 的不同实现，如 CriticalSectionDevice、ExclusiveDevice 等。我们将使用 ExclusiveDevice，因为它是从 SpiBus 获取 SpiDevice 的最简单方式，并且适用于没有其他设备共享 SPI 总线的情况。

### 为 RFID 读卡器设置 SPI

要与 RFID 模块通信，我们将使用 SPI2 外设初始化 SPI 实例。在此设置中，我们将 SPI 时钟配置为 5MHz，并将必要的引脚映射到 GPIO 以实现正确的通信。

```rust
let spi_bus = Spi::new(
    peripherals.SPI2,
    spi::master::Config::default()
        .with_frequency(Rate::from_mhz(5))
        .with_mode(spi::Mode::_0),
)
.unwrap()
.with_sck(peripherals.GPIO18)
.with_mosi(peripherals.GPIO23)
.with_miso(peripherals.GPIO19)
.into_async();

let sd_cs = Output::new(peripherals.GPIO5, Level::High, OutputConfig::default());
```

### 从 SPI 总线获取 `SpiDevice`

要使用 mfrc522 crate，我们需要一个 `SpiDevice`。由于我们只有来自 ESP-HAL 的 SPI 总线，我们将使用 embedded_hal_bus crate 从 SPI 总线获取 `SpiDevice`。

```rust
let delay = Delay::new();
let spi_dev = ExclusiveDevice::new(spi_bus, sd_cs, delay).unwrap();
```

### 初始化 mfrc522

接下来，我们初始化 MFRC522 驱动。为此，我们将 SpiDevice 实例用 mfrc522 crate 提供的 SpiInterface 包装器包裹，然后传递给 Mfrc522 初始化：

```rust
let spi_interface = SpiInterface::new(spi_dev);
let mut rfid = Mfrc522::new(spi_interface).init().unwrap();
```

### 辅助函数：将字节数组打印为十六进制字符串

我们将使用这个辅助函数将 u8 字节数组（如 UID）转换为可打印的十六进制字符串。这个函数将在整个 RFID 练习中用于显示数据。根据每个练习的具体需求，我们可能会稍微调整它。

```rust
fn print_hex_bytes(data: &[u8]) {
    for &b in data.iter() {
        print!("{:02x} ", b);
    }
    println!("");
}
```

### 读取 UID 并打印

读取 UID 的主逻辑很简单。我们持续发送 REQA（Request A）命令来检查附近是否有卡片或标签。如果卡片存在，它会以 ATQA（Answer To reQuest code A）响应。

ATQA 包含有关标签类型、能力和其他信息。利用 ATQA 响应，我们选择标签并获取其 UID。

```rust
loop {
    if let Ok(atqa) = rfid.reqa() {
        println!("Answer To reQuest code A");
        Timer::after(Duration::from_millis(50)).await;
        if let Ok(uid) = rfid.select(&atqa) {
            print_hex_bytes(uid.as_bytes());
            Timer::after(Duration::from_millis(500)).await;
        }
    }
}
```

## 克隆现有项目

你也可以克隆（或参考）我创建的项目，并导航到 `rfid-uid` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/rfid-uid
```

## 打印 UID

将代码烧录（flash）到 ESP32 后，将 RFID 标签靠近读卡器。UID 字节将以十六进制格式显示在系统控制台中。接下来，用钥匙扣标签（key fob）试试；它应该显示不同的 UID。

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

// SPI
use esp_hal::gpio::{Level, Output, OutputConfig};
use esp_hal::spi;
use esp_hal::spi::master::Spi;
use esp_hal::time::Rate;

// RFID Reader
use embedded_hal_bus::spi::ExclusiveDevice;
use esp_hal::delay::Delay;
use mfrc522::Mfrc522;
use mfrc522::comm::blocking::spi::SpiInterface;

use esp_println::{self as _, print, println};

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

    let spi_bus = Spi::new(
        peripherals.SPI2,
        spi::master::Config::default()
            .with_frequency(Rate::from_mhz(5))
            .with_mode(spi::Mode::_0),
    )
    .unwrap()
    .with_sck(peripherals.GPIO18)
    .with_mosi(peripherals.GPIO23)
    .with_miso(peripherals.GPIO19)
    .into_async();

    let sd_cs = Output::new(peripherals.GPIO5, Level::High, OutputConfig::default());

    let delay = Delay::new();
    let spi_dev = ExclusiveDevice::new(spi_bus, sd_cs, delay).unwrap();

    let spi_interface = SpiInterface::new(spi_dev);
    let mut rfid = Mfrc522::new(spi_interface).init().unwrap();

    loop {
        if let Ok(atqa) = rfid.reqa() {
            println!("Answer To reQuest code A");
            Timer::after(Duration::from_millis(50)).await;
            if let Ok(uid) = rfid.select(&atqa) {
                print_hex_bytes(uid.as_bytes());
                Timer::after(Duration::from_millis(500)).await;
            }
        }
    }
}

fn print_hex_bytes(data: &[u8]) {
    for &b in data.iter() {
        print!("{:02x} ", b);
    }
    println!("");
}
```

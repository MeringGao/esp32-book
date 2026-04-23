# 编写 Rust 代码从 SD 卡读取文件

让我们创建一个简单的程序，从 SD 卡读取文件并将其内容输出到系统控制台。确保 SD 卡已格式化为 FAT32，并且包含一个要读取的文件（例如，名为 "FERRIS.TXT"，内容为 "Hello, World!"）。

## 使用 esp-generate 生成项目

我们将为这个项目启用异步（Embassy）支持。要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 sdcard-read
```

这将打开一个选项选择界面。

- 选择 "Enable unstable HAL features" 选项
- 然后选择 "Adds embassy framework support" 选项

按键盘上的 "s" 保存即可。

## 需要的额外 Crate

更新你的 Cargo.toml，在现有依赖之外添加这些额外的 crate。

```toml
# SD 卡驱动
embedded-sdmmc = "0.9.0"
# 将 Spi 总线转换为 SpiDevice
embedded-hal-bus = "0.3.0"
```

### embedded-sdmmc

我们将使用这个 crate 在嵌入式设备上读写 SD 卡文件。它不使用 alloc 或集合，以保持较低的内存占用。

### embedded-hal-bus

要理解为什么我们需要 "embedded-hal-bus" crate，我们首先需要了解 Embedded HAL（Hardware Abstraction Layer，硬件抽象层）。Embedded HAL 提供了几个 trait，为微控制器上的通用外设（如 GPIO、PWM 和通信接口（如 I2C、SPI 和 UART））提供了标准的控制方式。这使得驱动可以兼容多种微控制器（例如 ESP32、Raspberry Pi Pico）。

当我们想使用 SPI 与 SD 卡通信时，embedded-hal 提供了 `SpiBus` 和 `SpiDevice` trait 来支持总线共享。SpiBus 代表整个总线，而 SpiDevice 代表该总线上的一个设备。微控制器专用的 HAL（例如 esp-hal）通常实现 SpiBus trait，而像 sdmmc 这样的设备驱动则实现 SpiDevice trait。

因此，我们需要从 SpiBus 获取 SpiDevice 以便与 SD 卡一起使用。这就是 embedded-hal-bus crate 发挥作用的地方。它提供了 SpiDevice 的不同实现，如 CriticalSectionDevice、ExclusiveDevice 等。我们将使用 ExclusiveDevice，因为它是从 SpiBus 获取 SpiDevice 的最简单方式，并且适用于没有其他设备共享 SPI 总线的情况。

## 虚拟时间源（Dummy Timesource）

当你在电脑上处理文件时，你可能会注意到文件和目录有创建和修改时间，用于跟踪何时进行了更改。SD 卡也是如此；当你创建或修改文件时，文件的时间戳会更新。

sdmmc 驱动需要一个时间源（time source）来获取当前时间以进行这些更新。它提供了一个 TimeSource trait，你需要实现它并在初始化期间传递给 VolumeManager。

由于我们只读取文件，不会使用此功能，因此我们将创建一个实现 TimeSource trait 的 DummyTimeSource。

```rust
/// 代码来自 https://github.com/rp-rs/rp-hal-boards/blob/main/boards/rp-pico/examples/pico_spi_sd_card.rs
/// 一个虚拟时间源，主要用于创建文件。
#[derive(Default)]
pub struct DummyTimesource();

impl TimeSource for DummyTimesource {
    // 理论上你可以在这里使用 rp2040 的 RTC，如果你有
    // 任何外部时间同步设备的话。
    fn get_timestamp(&self) -> Timestamp {
        Timestamp {
            year_since_1970: 0,
            zero_indexed_month: 0,
            zero_indexed_day: 0,
            hours: 0,
            minutes: 0,
            seconds: 0,
        }
    }
}
```

## 为 SD 卡读卡器设置 SPI

要与 SD 卡读卡器通信，我们将使用 SPI2 外设初始化 SPI 实例。SD 卡要求 SPI 时钟在 100 kHz 到 400 kHz 之间运行。在此设置中，我们将 SPI 时钟配置为 400 kHz，并将必要的引脚映射到 GPIO 以实现正确的通信。

SCK（Serial Clock，串行时钟）将分配给 GPIO18，MOSI（Master Out, Slave In，主出从入）分配给 GPIO23，MISO（Master In, Slave Out，主入从出）分配给 GPIO19。此外，我们将在 GPIO5 上配置 CS（Chip Select，片选）引脚，并将其初始状态设置为高电平。

```rust
let spi_bus = Spi::new(
    peripherals.SPI2,
    spi::master::Config::default()
        .with_frequency(Rate::from_khz(400))
        .with_mode(spi::Mode::_0),
)
.unwrap()
.with_sck(peripherals.GPIO18)
.with_mosi(peripherals.GPIO23)
.with_miso(peripherals.GPIO19)
.into_async();

let sd_cs = Output::new(peripherals.GPIO5, Level::High, OutputConfig::default());
```

SPI 配置完成后，我们将为 SD 卡读卡器创建一个 SpiDevice。为此，我们将使用 embedded-hal-bus crate 提供的 ExclusiveDevice。最后，我们初始化 SD 卡。

```rust
let spi_dev = ExclusiveDevice::new(spi_bus, sd_cs, Delay).unwrap();
let sdcard = SdCard::new(spi_dev, Delay);
```

## 打印 SD 卡容量

接下来，我们将使用卷管理器（volume manager）以字节为单位获取 SD 卡的容量并打印出来。

```rust
println!("Init SD card controller and retrieve card size...");
let sd_size = sdcard.num_bytes().unwrap();
println!("card size is {} bytes\r\n", sd_size);
```

## 卷管理器

下一步是初始化卷管理器以处理 SD 卡上的分区和文件系统。

```rust
// 现在让我们在块设备上查找卷（也称为分区）。
// 为此，我们需要一个卷管理器。它将取得块设备的所有权。
let volume_mgr = VolumeManager::new(sdcard, DummyTimesource::default());
```

## 打开目录

我们使用卷管理器打开 SD 卡上的第一个主分区（VolumeIdx(0)）。之后，我们访问该分区的根目录。

```rust
// 尝试访问卷 0（即第一个分区）。
// 卷对象保存了该卷上文件系统的信息。
let volume0 = volume_mgr.open_volume(VolumeIdx(0)).unwrap();

let root_dir = volume0.open_root_dir().unwrap();
```

### 打开文件

让我们以只读模式打开 "FERRIS.TXT" 文件并读取它。确保你之前已经从系统中将此文件添加到了 SD 卡中，并包含一些内容。

```rust
let mut my_file = root_dir
    .open_file_in_dir("FERRIS.TXT", embedded_sdmmc::Mode::ReadOnly)
    .unwrap();
```

### 打印文件内容

我们将读取文件直到到达末尾。我们将每个字节转换为字符并打印它。

```rust
while !my_file.is_eof() {
    let mut buffer = [0u8; 32];

    if let Ok(n) = my_file.read(&mut buffer) {
        for b in &buffer[..n] {
            print!("{}", *b as char);
        }
    }
}
```

## 克隆已有项目

你也可以克隆（或参考）我创建的项目，并导航到 `sdcard-read` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/sdcard-read
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
use embassy_time::{Delay, Duration, Timer};
use embedded_hal_bus::spi::ExclusiveDevice;
use esp_hal::gpio::{Level, Output, OutputConfig};
use esp_hal::spi;
use esp_hal::time::Rate;
use esp_hal::timer::timg::TimerGroup;
use esp_hal::{clock::CpuClock, spi::master::Spi};
use esp_println::{self as _, print, println};

// SD 卡读卡器
use embedded_sdmmc::{SdCard, TimeSource, Timestamp, VolumeIdx, VolumeManager};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

// 创建 ESP-IDF 引导加载程序所需的应用描述符。
// 更多信息请参阅：<https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
esp_bootloader_esp_idf::esp_app_desc!();

/// 代码来自 https://github.com/rp-rs/rp-hal-boards/blob/main/boards/rp-pico/examples/pico_spi_sd_card.rs
/// 一个虚拟时间源，主要用于创建文件。
#[derive(Default)]
pub struct DummyTimesource();

impl TimeSource for DummyTimesource {
    // 理论上你可以在这里使用 rp2040 的 RTC，如果你有
    // 任何外部时间同步设备的话。
    fn get_timestamp(&self) -> Timestamp {
        Timestamp {
            year_since_1970: 0,
            zero_indexed_month: 0,
            zero_indexed_day: 0,
            hours: 0,
            minutes: 0,
            seconds: 0,
        }
    }
}

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
            .with_frequency(Rate::from_khz(400))
            .with_mode(spi::Mode::_0),
    )
    .unwrap()
    .with_sck(peripherals.GPIO18)
    .with_mosi(peripherals.GPIO23)
    .with_miso(peripherals.GPIO19)
    .into_async();

    let sd_cs = Output::new(peripherals.GPIO5, Level::High, OutputConfig::default());
    let spi_dev = ExclusiveDevice::new(spi_bus, sd_cs, Delay).unwrap();
    let sdcard = SdCard::new(spi_dev, Delay);

    println!("Init SD card controller and retrieve card size...");
    let sd_size = sdcard.num_bytes().unwrap();
    println!("card size is {} bytes\r\n", sd_size);

    // 现在让我们在块设备上查找卷（也称为分区）。
    // 为此，我们需要一个卷管理器。它将取得块设备的所有权。
    let volume_mgr = VolumeManager::new(sdcard, DummyTimesource::default());

    // 尝试访问卷 0（即第一个分区）。
    // 卷对象保存了该卷上文件系统的信息。
    let volume0 = volume_mgr.open_volume(VolumeIdx(0)).unwrap();

    let root_dir = volume0.open_root_dir().unwrap();

    let my_file = root_dir
        .open_file_in_dir("FERRIS.TXT", embedded_sdmmc::Mode::ReadOnly)
        .unwrap();

    while !my_file.is_eof() {
        let mut buffer = [0u8; 32];

        if let Ok(n) = my_file.read(&mut buffer) {
            for b in &buffer[..n] {
                print!("{}", *b as char);
            }
        }
    }

    loop {
        Timer::after(Duration::from_secs(30)).await;
    }
}
```

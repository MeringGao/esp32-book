# 使用 ESP32 将文件写入 SD 卡的 Rust 代码

在本练习中，我们将在 SD 卡上创建（或覆盖）一个文件，并向其中写入 "Hello, Ferris!"。

## 使用 esp-generate 生成项目

我们将为这个项目启用异步（Embassy）支持。要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 sdcard-write
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

## 用于时间解析
jiff = { version = "0.2.16", default-features = false, features = ["static"] }
```

## 带 RTC 的 TimeSource

在深入写入过程之前，让我们为 sdmmc crate 创建一个 TimeSource。在上一个练习中，我们只读取文件并使用了一个虚拟时间源。然而，在本练习中，我们想要创建文件并确保其时间元数据得到相应更新。为此，我们将创建一个足够接近实时的时间源。

我们将为此目的使用板载 RTC（Real-Time Clock，实时时钟）。虽然这不是一个完美的解决方案；因为 RTC 需要设置初始时间（我们将通过环境变量传递）。每当 ESP32 重新启动时，它也会重置。或者，你可以使用 Wi-Fi 连接从 NTP 服务器获取当前时间。

我们将创建一个实现 TimeSource trait 的结构体 SdTimeSource，并使用 Rtc 获取当前时间。我们必须指定自 1970 年（Unix 纪元）以来经过了多少年。我们将通过当前年份减去 1970 来获得。我们还需要以零索引格式指定月份和日期信息，以及时间本身。

```rust
static TZ: jiff::tz::TimeZone = jiff::tz::get!("America/New_York");

struct SdTimeSource {
    timer: Rtc<'static>,
}

impl SdTimeSource {
    fn new(timer: Rtc<'static>) -> Self {
        Self { timer }
    }

    fn current_time(&self) -> u64 {
        self.timer.current_time_us()
    }
}

static TZ: jiff::tz::TimeZone = jiff::tz::get!("America/New_York");

impl TimeSource for SdTimeSource {
    fn get_timestamp(&self) -> Timestamp {
        let now_us = self.current_time();

        // 转换为 jiff Time
        let now = jiff::Timestamp::from_microsecond(now_us as i64).unwrap();
        let now = now.to_zoned(TZ.clone());

        Timestamp {
            year_since_1970: (now.year() - 1970).unsigned_abs() as u8,
            zero_indexed_month: now.month().wrapping_sub(1) as u8,
            zero_indexed_day: now.day().wrapping_sub(1) as u8,
            hours: now.hour() as u8,
            minutes: now.minute() as u8,
            seconds: now.second() as u8,
        }
    }
}
```

接下来，我们将更新 SD 卡初始化代码以使用 SdTimeSource 作为时间源。我们将通过环境变量传递当前时间，使用 jiff 解析它，并将其设置为 RTC 的当前时间。

```rust
// 用于 SD 卡的定时器
let rtc = Rtc::new(peripherals.LPWR);
let current_time_us: u64 = env!("CURRENT_TIME_US")
    .parse()
    .expect("Invalid microseconds");
rtc.set_current_time_us(current_time_us);

let sd_timer = SdTimeSource::new(rtc);

println!("Init SD card controller and retrieve card size...");
let sd_size = sdcard.num_bytes().unwrap();
println!("card size is {} bytes\r\n", sd_size);

// 现在让我们在块设备上查找卷（也称为分区）。
// 为此，我们需要一个卷管理器。它将取得块设备的所有权。
let volume_mgr = VolumeManager::new(sdcard, sd_timer);

// 尝试访问卷 0（即第一个分区）。
// 卷对象保存了该卷上文件系统的信息。
let volume0 = volume_mgr.open_volume(VolumeIdx(0)).unwrap();

let root_dir = volume0.open_root_dir().unwrap();
```

## 写入文件

让我们以 ReadWriteCreateOrTruncate 模式打开文件。如果文件不存在，这将创建它。如果文件已存在，它将截断文件，清除任何现有内容。

```rust
let mut my_file = root_dir
.open_file_in_dir(
    "FERRIS.TXT",
    embedded_sdmmc::Mode::ReadWriteCreateOrTruncate,
)
.unwrap();
```

文件打开后，我们可以将消息写入其中，然后刷新（flush）以确保数据被保存。

```rust
let line = "Hello, Ferris!";
if let Ok(()) = my_file.write(line.as_bytes()) {
    my_file.flush().unwrap();
    println!("Written Data");
} else {
    println!("Not wrote");
}
```

要验证，你可以将 SD 卡连接到你的电脑，或者重新运行我们之前使用的 SD 卡读取程序。

## 克隆已有项目

你也可以克隆（或参考）我创建的项目，并导航到 `sdcard-write` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/sdcard-write
```

## 如何运行？

我们将当前日期和时间作为环境变量传递。为此，我们将在 cargo run 之前添加一个 shell 命令，获取当前日期和时间并将其传递给 cargo run。这在 Fish 等 shell 或 Windows 上可能无法工作。

```sh
CURRENT_DATETIME="$(date '+%Y-%m-%d %H:%M:%S')" cargo run --release
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
use esp_hal::clock::CpuClock;
use esp_hal::timer::timg::TimerGroup;
use esp_println::{self as _, println};

// SPI
use embedded_hal_bus::spi::ExclusiveDevice;
use esp_hal::gpio::{Level, Output, OutputConfig};
use esp_hal::spi::{self, master::Spi};
use esp_hal::time::Rate;

// SD 卡读卡器
use embedded_sdmmc::{SdCard, TimeSource, Timestamp, VolumeIdx, VolumeManager};

// 用于时间
use esp_hal::rtc_cntl::Rtc;

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

// 创建 ESP-IDF 引导加载程序所需的应用描述符。
// 更多信息请参阅：<https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
esp_bootloader_esp_idf::esp_app_desc!();

struct SdTimeSource {
    timer: Rtc<'static>,
}

impl SdTimeSource {
    fn new(timer: Rtc<'static>) -> Self {
        Self { timer }
    }

    fn current_time(&self) -> u64 {
        self.timer.current_time_us()
    }
}

static TZ: jiff::tz::TimeZone = jiff::tz::get!("America/New_York");

impl TimeSource for SdTimeSource {
    fn get_timestamp(&self) -> Timestamp {
        let now_us = self.current_time();

        // 转换为 jiff Time
        let now = jiff::Timestamp::from_microsecond(now_us as i64).unwrap();
        let now = now.to_zoned(TZ.clone());

        Timestamp {
            year_since_1970: (now.year() - 1970).unsigned_abs() as u8,
            zero_indexed_month: now.month().wrapping_sub(1) as u8,
            zero_indexed_day: now.day().wrapping_sub(1) as u8,
            hours: now.hour() as u8,
            minutes: now.minute() as u8,
            seconds: now.second() as u8,
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

    // 用于 SD 卡的定时器
    let rtc = Rtc::new(peripherals.LPWR);
    let current_time_us: u64 = env!("CURRENT_TIME_US")
        .parse()
        .expect("Invalid microseconds");
    rtc.set_current_time_us(current_time_us);

    let sd_timer = SdTimeSource::new(rtc);

    let sdcard = SdCard::new(spi_dev, Delay);

    println!("Init SD card controller and retrieve card size...");
    let sd_size = sdcard.num_bytes().unwrap();
    println!("card size is {} bytes\r\n", sd_size);

    // 现在让我们在块设备上查找卷（也称为分区）。
    // 为此，我们需要一个卷管理器。它将取得块设备的所有权。
    let volume_mgr = VolumeManager::new(sdcard, sd_timer);

    // 尝试访问卷 0（即第一个分区）。
    // 卷对象保存了该卷上文件系统的信息。
    let volume0 = volume_mgr.open_volume(VolumeIdx(0)).unwrap();

    let root_dir = volume0.open_root_dir().unwrap();

    let my_file = root_dir
        .open_file_in_dir(
            "FERRIS.TXT",
            embedded_sdmmc::Mode::ReadWriteCreateOrTruncate,
        )
        .unwrap();

    let line = "Hello, Ferris!";
    if let Ok(()) = my_file.write(line.as_bytes()) {
        my_file.flush().unwrap();
        println!("Written Data");
    } else {
        println!("Not wrote");
    }

    loop {
        Timer::after(Duration::from_secs(30)).await;
    }
}
```

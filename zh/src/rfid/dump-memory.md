## 转储全部存储内容

你已经学会了如何通过认证读取第一个扇区（sector 0）每个块中的数据。现在，我们将遍历每个扇区。每次移动到新扇区时都需要重新认证。对于每个扇区，我们将显示每个 4 块中的 16 字节数据。

为了更清晰，我们会添加一些格式和标签，指示我们当前引用的是哪个扇区和块（包括绝对块号和相对于扇区的块号），以及该块是扇区尾部块（sector trailer）还是数据块（data block）。

> 你可以在之前章节创建的项目基础上继续工作，也可以创建一个新项目，按照相同的步骤操作，然后继续执行以下说明。

### 遍历扇区

我们将创建一个单独的函数来遍历所有 16 个扇区（扇区 0 到 15），读取每个扇区中的所有块，并打印它们的数据。

```rust
fn dump_memory<E, COMM: mfrc522::comm::Interface<Error = E>>(
    uid: &mfrc522::Uid,
    rfid: &mut Mfrc522<COMM, mfrc522::Initialized>,
) {
    for sector in 0..16 {
        // 打印扇区号
        println!("\n\n-----------SECTOR {}-----------", sector);
        read_sector(uid, sector, rfid);
    }
}
```

### 标签

`read_sector` 函数遵循与之前相同的逻辑，但添加了格式和标签。它现在会打印绝对块号、相对于扇区的块号，以及制造商数据（MFD）块和扇区尾部块的标签。

```rust
fn read_sector<E, COMM: mfrc522::comm::Interface<Error = E>>(
    uid: &mfrc522::Uid,
    sector: u8,
    rfid: &mut Mfrc522<COMM, mfrc522::Initialized>,
) {
    const AUTH_KEY: [u8; 6] = [0xFF; 6];

    let block_offset = sector * 4;
    rfid.mf_authenticate(uid, block_offset, &AUTH_KEY)
        .map_err(|_| "Auth failed")
        .unwrap();

    for abs_block in block_offset..block_offset + 4 {
        let rel_block = abs_block - block_offset;
        let data = rfid.mf_read(abs_block).map_err(|_| "Read failed").unwrap();

        // 打印块的绝对编号和相对编号
        print!("\nBLOCK {} (REL: {}) | ", abs_block, rel_block);
        print_hex_bytes(&data);

        // 打印块类型
        let block_type = get_block_type(sector, rel_block);
        print!("| {} ", block_type);
    }
}
```

我们将创建一个小型辅助函数，根据扇区及其相对块号来确定块类型。

```rust
const fn get_block_type(sector: u8, rel_block: u8) -> &'static str {
    match rel_block {
        0 if sector == 0 => "MFD",
        3 => "TRAILER",
        _ => "DATA",
    }
}
```

### 主循环

主循环没有太大变化。我们只是调用 `dump_memory` 函数而不是 `read_sector`。

```rust
loop {
    if let Ok(atqa) = rfid.reqa() {
        println!("Got atqa");
        Timer::after(Duration::from_millis(50)).await;
        if let Ok(uid) = rfid.select(&atqa) {
            dump_memory(&uid, &mut rfid);
            rfid.hlta().unwrap();
            rfid.stop_crypto1().unwrap();
        }
    }
}
```

## 克隆现有项目

你也可以克隆（或参考）我创建的项目，并导航到 `rfid-dump` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/rfid-dump
```

## 转储结果

当你运行程序并将标签或钥匙扣标签（key fob）靠近时，你应该会看到类似这样的输出。如果你注意到块 18（扇区 4 的块 2）中的 0x40..0x43 字节并想知道为什么它们在那里；观察得很仔细！那是我写入标签的自定义数据。

<img style="display: block; margin: auto;" src="./images/rfid-dump.png"/>

图片只显示了前 5 个扇区。当你运行程序时，你应该能看到所有 16 个扇区的数据。

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
            println!("Got atqa");
            Timer::after(Duration::from_millis(50)).await;
            if let Ok(uid) = rfid.select(&atqa) {
                dump_memory(&uid, &mut rfid);
                rfid.hlta().unwrap();
                rfid.stop_crypto1().unwrap();
            }
        }
    }
}

fn read_sector<E, COMM: mfrc522::comm::Interface<Error = E>>(
    uid: &mfrc522::Uid,
    sector: u8,
    rfid: &mut Mfrc522<COMM, mfrc522::Initialized>,
) {
    const AUTH_KEY: [u8; 6] = [0xFF; 6];

    let block_offset = sector * 4;
    rfid.mf_authenticate(uid, block_offset, &AUTH_KEY)
        .map_err(|_| "Auth failed")
        .unwrap();

    for abs_block in block_offset..block_offset + 4 {
        let rel_block = abs_block - block_offset;
        let data = rfid.mf_read(abs_block).map_err(|_| "Read failed").unwrap();

        // 打印块的绝对编号和相对编号
        print!("\nBLOCK {} (REL: {}) | ", abs_block, rel_block);
        print_hex_bytes(&data);

        // 打印块类型
        let block_type = get_block_type(sector, rel_block);
        print!("| {} ", block_type);
    }
}

fn print_hex_bytes(data: &[u8]) {
    for &b in data.iter() {
        print!("{:02x} ", b);
    }
}

const fn get_block_type(sector: u8, rel_block: u8) -> &'static str {
    match rel_block {
        0 if sector == 0 => "MFD",
        3 => "TRAILER",
        _ => "DATA",
    }
}

fn dump_memory<E, COMM: mfrc522::comm::Interface<Error = E>>(
    uid: &mfrc522::Uid,
    rfid: &mut Mfrc522<COMM, mfrc522::Initialized>,
) {
    for sector in 0..16 {
        // 打印扇区号
        println!("\n\n-----------SECTOR {}-----------", sector);

        read_sector(uid, sector, rfid);
    }
}
```

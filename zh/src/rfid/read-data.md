# 读取数据

在本节中，我们将读取第一个扇区（sector 0）中的所有块。正如我们之前提到的，要读取或写入 RFID 标签上的特定块，首先需要认证（authenticate）对应的扇区。

## 创建项目

按照之前的相同步骤，添加依赖并完成 RFID 组件的基础设置。这次，我们将修改代码来读取数据。

## 认证

大多数标签都带有默认密钥，通常是 6 个 0xFF。你可能需要查阅文档来找到默认密钥，或者尝试其他常见密钥。对于我们使用的 RFID 读卡器，默认密钥是 6 个 0xFF。

认证需要：
- 标签的 UID（通过 REQA 和 Select 命令获取）。
- 扇区内的块号。
- 密钥（本例中硬编码）。

## 读取块

在从块中读取数据之前，我们必须先对该块进行认证。如果读取操作成功，函数将返回该块的 16 字节数据。第一个扇区（sector 0）由 4 个块组成，绝对块号范围为 0 到 3。对于更高的扇区，绝对块号相应增加（例如，扇区 1 的块为 4、5、6、7）。

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
        let data = rfid.mf_read(abs_block).map_err(|_| "Read failed").unwrap();
        print_hex_bytes(&data);
    }
}
```

在这个函数中，我们将认证密钥硬编码为 0xFF 值的字节数组。接下来，我们通过将选定的扇区乘以 4 来计算块偏移量。例如，如果我们选择扇区 2，乘以 4 得到块偏移量为 8。然后我们将 UID、块偏移量和认证密钥传递给认证函数，以获取对该扇区的访问权限。

认证成功后，我们从块偏移量开始循环遍历扇区内的 4 个块。对于每个块，我们读取数据并将其内容以十六进制字符串形式打印出来。

## 主循环

在主循环中，检测到 RFID 标签（即接收到 ATQA）并选择标签进行进一步通信后，我们调用 read_sector 函数。这里，我们指定读取扇区 0。一旦块数据读取完毕，我们将发送 HLTA 和 stop_crypto1 命令，将卡片置于 HALT 状态。

```rust
let sector_num = 0;
loop {
    if let Ok(atqa) = rfid.reqa() {
        println!("Got atqa");
        Timer::after(Duration::from_millis(50)).await;
        if let Ok(uid) = rfid.select(&atqa) {
            println!("Reading sector: {}", sector_num);
            read_sector(&uid, sector_num, &mut rfid);
            rfid.hlta().unwrap();
            rfid.stop_crypto1().unwrap();
        }
    }
}
```

## 克隆现有项目

你也可以克隆（或参考）我创建的项目，并导航到 `rfid-read` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/rfid-read
```

## 打印第一个扇区

将 RFID 标签靠近读卡器，系统控制台将显示从第一个扇区（sector 0）的块中读取的数据字节。

<img style="display: block; margin: auto;" src="./images/rfid-read-block-0.png"/>

其中显示 "13 73 73 31" 的位置，将显示你的 RFID 标签的 UID。

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

    let sector_num = 0;
    loop {
        if let Ok(atqa) = rfid.reqa() {
            println!("Got atqa");
            Timer::after(Duration::from_millis(50)).await;
            if let Ok(uid) = rfid.select(&atqa) {
                println!("Reading sector: {}", sector_num);
                read_sector(&uid, sector_num, &mut rfid);
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
        let data = rfid.mf_read(abs_block).map_err(|_| "Read failed").unwrap();
        print_hex_bytes(&data);
    }
}

fn print_hex_bytes(data: &[u8]) {
    for &b in data.iter() {
        print!("{:02x} ", b);
    }
    println!("");
}
```

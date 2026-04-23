## 更改认证密钥

让我们更改扇区 1 的认证密钥（KeyA）。默认情况下，它设置为 `FF FF FF FF FF FF`。我们将把它更新为 `52 75 73 74 65 64`，这是 "Rusted" 的十六进制表示。为此，我们需要修改扇区 1 的尾部块（block 3），同时保持扇区的其余部分不变。

在继续之前，最好先验证该块的当前内容。运行 [转储存储内容](./dump-memory.md) 或 [读取数据](./read-data.md) 程序来检查。

> [!Note]
> MIFARE Classic 1K 卡预配置的默认密钥为 FF FF FF FF FF FF，KeyA 和 KeyB 均如此。读取尾部块时，KeyA 的值会全部返回为零（00 00 00 00 00 00），而 KeyB 则按原值返回。

我们还将修改 KeyB 的内容，以验证写入是否成功。我们将 KeyB 设置为 "Ferris" 的十六进制字节（46 65 72 72 69 73）。

在写入之前，你的块中的访问字节和 KeyB 值应该与我提供的大致相同，但仔细检查总是比猜测更好。

计划如下：
1. 在程序中，我们将默认密钥（`FF FF FF FF FF FF`）硬编码到一个名为 `current_key` 的变量中。
2. 将 `new_key` 设置为 `Rusted`（十六进制字节）。这是为了在写入后打印块内容所必需的；否则，我们会得到认证错误。
3. 程序将在写入前后都打印块的内容。

一旦密钥更新，再次将标签靠近。你很可能会看到 "Auth failed" 错误。如果你想知道为什么，恭喜你——你发现了！新密钥已成功写入，所以硬编码的 `current_key` 不再有效。要验证，请修改 `read-data` 程序以使用新密钥（`Rusted`），然后再试一次。

### 密钥和数据

DATA 数组包含新的 KeyA（"Rusted" 的十六进制）、访问位和 KeyB（"Ferris" 的十六进制）。current_key 设置为默认的 FF FF FF FF FF FF，new_key 是 DATA 的前 6 个字节，即 "Rusted"。

```rust
let target_sector = 1;
let rel_block = 3; // 扇区内的相对块号（扇区 1 的第 4 个块）
const DATA: [u8; 16] = [
    0x52, 0x75, 0x73, 0x74, 0x65, 0x64, // Key A: "Rusted"
    0xFF, 0x07, 0x80, 0x69, // 访问位和尾部字节
    0x46, 0x65, 0x72, 0x72, 0x69, 0x73, // Key B: "Ferris"
];
let current_key = &[0xFF; 6];
let new_key: &[u8; 6] = &DATA[..6].try_into().unwrap(); // 块的前 6 个字节

```

### 写入块函数

我们对 write_block 函数做了轻微修改，使其接受密钥作为参数。

```rust
fn write_block<E, COMM: mfrc522::comm::Interface<Error = E>>(
    uid: &mfrc522::Uid,
    sector: u8,
    rel_block: u8,
    data: [u8; 16],
    rfid: &mut Mfrc522<COMM, mfrc522::Initialized>,
    auth_key: &[u8; 6], // 用于认证的额外参数
) {
    let block_offset = sector * 4;
    let abs_block = block_offset + rel_block;

    rfid.mf_authenticate(uid, block_offset, auth_key)
        .map_err(|_| "Auth failed")
        .unwrap();

    rfid.mf_write(abs_block, data)
        .map_err(|_| "Write failed")
        .unwrap();
}
```

### 读取扇区函数

我们对 read_sector 函数也做了类似的修改。

```rust
fn read_sector<E, COMM: mfrc522::comm::Interface<Error = E>>(
    uid: &mfrc522::Uid,
    sector: u8,
    rfid: &mut Mfrc522<COMM, mfrc522::Initialized>,
    auth_key: &[u8; 6], // 用于认证的额外参数
) {
    let block_offset = sector * 4;
    rfid.mf_authenticate(uid, block_offset, auth_key)
        .map_err(|_| "Auth failed")
        .unwrap();

    for abs_block in block_offset..block_offset + 4 {
        let data = rfid.mf_read(abs_block).map_err(|_| "Read failed").unwrap();
        print_hex_bytes(&data);
    }
}
```

### 主循环

主循环中没有新内容。所有读写函数都是你之前见过的。我们只是打印更改密钥前后的扇区内容。

```rust
loop {
    if let Ok(atqa) = rfid.reqa() {
        println!("Got atqa");
        Timer::after(Duration::from_millis(50)).await;
        if let Ok(uid) = rfid.select(&atqa) {
            println!("\r\n----写入前----");
            read_sector(&uid, target_sector, &mut rfid, current_key);

            write_block(&uid, target_sector, rel_block, DATA, &mut rfid, current_key);

            println!("\r\n----写入后----");
            read_sector(&uid, target_sector, &mut rfid, new_key);
            rfid.hlta().unwrap();
            rfid.stop_crypto1().unwrap();
        }
    }
}
```

## 克隆现有项目

你也可以克隆（或参考）我创建的项目，并导航到 `rfid-change-key` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/rfid-change-key
```

## 输出

如你在输出中所见，当你运行程序时，它会显示目标块在写入前后的内容。更改密钥后，将标签再次靠近读卡器会显示 "auth failed" 消息，因为 current_key 已被更改；新密钥是 52 75 73 74 65 64（Rusted）。

<img style="display: block; margin: auto;" src="./images/change-auth-key.png"/>

你也可以修改之前使用的读取数据程序，使用新密钥来验证。

**注意：** 如果你想知道为什么 Key A 没有显示，正如我们之前解释的，如果你使用 Key A 进行认证，就无法读取 Key A 的值。更多详情请参考 [访问位](./access-bits.md) 章节。

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

    let target_sector = 1;
    let rel_block = 3; // 扇区内的相对块号（扇区 1 的第 4 个块）
    const DATA: [u8; 16] = [
        0x52, 0x75, 0x73, 0x74, 0x65, 0x64, // Key A: "Rusted"
        0xFF, 0x07, 0x80, 0x69, // 访问位和尾部字节
        0x46, 0x65, 0x72, 0x72, 0x69, 0x73, // Key B: "Ferris"
    ];
    let current_key = &[0xFF; 6];
    // 如果要重置为 0xFF，取消下面注释
    // const DATA: [u8; 16] = [
    //     0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, // Key A: "Rusted"
    //     0xFF, 0x07, 0x80, 0x69, // 访问位和尾部字节
    //     0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, // Key B: "Ferris"
    // ];
    // let current_key = &[0x52, 0x75, 0x73, 0x74, 0x65, 0x64];
    let new_key: &[u8; 6] = &DATA[..6].try_into().unwrap(); // 块的前 6 个字节

    loop {
        if let Ok(atqa) = rfid.reqa() {
            println!("Got atqa");
            Timer::after(Duration::from_millis(50)).await;
            if let Ok(uid) = rfid.select(&atqa) {
                println!("\r\n----Before Write----");
                read_sector(&uid, target_sector, &mut rfid, current_key);

                write_block(&uid, target_sector, rel_block, DATA, &mut rfid, current_key);

                println!("\r\n----After Write----");
                read_sector(&uid, target_sector, &mut rfid, new_key);
                rfid.hlta().unwrap();
                rfid.stop_crypto1().unwrap();
            }
        }
    }
}

fn write_block<E, COMM: mfrc522::comm::Interface<Error = E>>(
    uid: &mfrc522::Uid,
    sector: u8,
    rel_block: u8,
    data: [u8; 16],
    rfid: &mut Mfrc522<COMM, mfrc522::Initialized>,
    auth_key: &[u8; 6], // 用于认证的额外参数
) {
    let block_offset = sector * 4;
    let abs_block = block_offset + rel_block;

    rfid.mf_authenticate(uid, block_offset, auth_key)
        .map_err(|_| "Auth failed")
        .unwrap();

    rfid.mf_write(abs_block, data)
        .map_err(|_| "Write failed")
        .unwrap();
}

fn read_sector<E, COMM: mfrc522::comm::Interface<Error = E>>(
    uid: &mfrc522::Uid,
    sector: u8,
    rfid: &mut Mfrc522<COMM, mfrc522::Initialized>,
    auth_key: &[u8; 6], // 用于认证的额外参数
) {
    let block_offset = sector * 4;
    rfid.mf_authenticate(uid, block_offset, auth_key)
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

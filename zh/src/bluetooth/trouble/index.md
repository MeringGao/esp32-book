# 如何在 ESP32 上使用蓝牙低功耗（BLE）编写 Rust 代码

让我们创建一个简单的程序来演示蓝牙低功耗（Bluetooth Low Energy, BLE）。在这个练习中，我们将使用 `Trouble` crate 和 Embassy。我们将定义一个包含两个特征的 GATT 服务：

- 一个特征支持读取和写入操作。
- 另一个特征仅支持读取操作。

我已经为这些属性（服务和特征）生成了 UUID，你可以使用相同的。

> 参考 Trouble 仓库获取更多示例：[https://github.com/embassy-rs/trouble/tree/main/examples](https://github.com/embassy-rs/trouble/tree/main/examples)

## 连接到 ESP32 蓝牙

要与 ESP32 的蓝牙交互，我们将使用 nRF Connect for Mobile 应用：

🔗 [nRF Connect for Mobile](https://www.nordicsemi.com/Products/Development-tools/nRF-Connect-for-mobile)

该应用允许我们读取和写入 ESP32 提供的数据。

## 使用 esp-generate 生成项目

我们将为此项目启用 async（Embassy）支持。要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 bluetooth-low-energy
```

这将打开一个屏幕，要求你选择选项。

- 首先，选择 "Enable unstable HAL features"。
- 选择 "Enable allocations via the esp-alloc crate"。
- 选择 "Adds embassy framework support"。
- 现在，你可以启用 "Enable BLE via the esp-radio crate (embassy-trouble)。"

按键盘上的 "s" 保存即可。

## 更新依赖

```toml
embassy-futures = "0.1.1"

# gatt_server 宏工作所需
embassy-sync = { version = "0.7" }
```

## 初始化 Wi-Fi 控制器

ESP32 共享一个射频用于 Wi-Fi 和蓝牙。为了初始化蓝牙，我们将使用与 Wi-Fi 相同的 Wi-Fi 控制器。

```rust
let radio_init = esp_radio::init().expect("Failed to initialize Wi-Fi/BLE controller");
```

让我们初始化蓝牙连接器。

```rust
// 更多示例请参阅 https://github.com/embassy-rs/trouble/tree/main/examples/esp32
let transport = BleConnector::new(&radio_init, peripherals.BT, Default::default()).unwrap();
let ble_controller = ExternalController::<_, 20>::new(transport);

// let mut resources: HostResources<DefaultPacketPool, CONNECTIONS_MAX, L2CAP_CHANNELS_MAX> =
//     HostResources::new();
// let _stack = trouble_host::new(ble_controller, &mut resources);
```

创建 BLE HCI 控制器后，我们将调用稍后会定义的 `run` 函数来启动 BLE 协议栈。

```rust
ble::run(ble_controller).await;

loop {
    Timer::after(Duration::from_secs(5)).await;
}
```

## main.rs 的完整代码

这是 `main.rs` 文件的完整代码。接下来，我们将创建一个名为 `ble` 的模块，在其中实现 run 函数以及用于广播和处理事件的辅助函数。

```rust
#![no_std]
#![no_main]
#![deny(
    clippy::mem_forget,
    reason = "mem::forget is generally not safe to do with esp_hal types, especially those \
    holding buffers for the duration of a data transfer."
)]

use bt_hci::controller::ExternalController;
use defmt::info;
use embassy_executor::Spawner;
use embassy_time::{Duration, Timer};
use esp_hal::clock::CpuClock;
use esp_hal::timer::timg::TimerGroup;
use esp_println as _;
use esp_radio::ble::controller::BleConnector;

// 我们的模块
use bluetooth_low_energy as lib;
use lib::ble;

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

extern crate alloc;

// 创建一个 esp-idf 引导加载程序所需的默认应用描述符。
// 更多信息请参阅：
// <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
// This creates a default app-descriptor required by the esp-idf bootloader.
// For more information see: <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
esp_bootloader_esp_idf::esp_app_desc!();

#[esp_rtos::main]
async fn main(_spawner: Spawner) -> ! {
    // generator version: 1.0.0

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    esp_alloc::heap_allocator!(#[unsafe(link_section = ".dram2_uninit")] size: 98767);

    let timg0 = TimerGroup::new(peripherals.TIMG0);
    esp_rtos::start(timg0.timer0);

    info!("Embassy initialized!");

    let radio_init = esp_radio::init().expect("Failed to initialize Wi-Fi/BLE controller");
    // 更多示例请参阅 https://github.com/embassy-rs/trouble/tree/main/examples/esp32
    let transport = BleConnector::new(&radio_init, peripherals.BT, Default::default()).unwrap();
    let ble_controller = ExternalController::<_, 20>::new(transport);
    // let mut resources: HostResources<DefaultPacketPool, CONNECTIONS_MAX, L2CAP_CHANNELS_MAX> =
    //     HostResources::new();
    // let _stack = trouble_host::new(ble_controller, &mut resources);

    ble::run(ble_controller).await;

    loop {
        Timer::after(Duration::from_secs(5)).await;
    }
}
```

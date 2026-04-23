# BLE 模块

在这个模块中，我们将定义 ESP32 发送和接收的数据类型，设置并运行服务器，以及处理连接。我们从这个官方 Trouble 仓库示例中借鉴并做了一些小的修改。你可以在[仓库](https://github.com/embassy-rs/trouble/tree/main/examples/apps)中找到更多示例。

让我们通过更新 lib.rs 文件来创建 BLE 模块：

```rust
#![no_std]
pub mod ble;
```

从现在开始，我们将在 `ble.rs` 模块文件中工作。让我们先添加所有必要的导入：

```rust
use defmt::{info, warn};

use embassy_futures::join::join;
use embassy_futures::select::select;

use embassy_time::Timer;

// BLE:
use trouble_host::prelude::*;
```

让我们为 BLE 协议栈定义最大并发连接数和 L2CAP 通道数：

```rust
/// 最大连接数
const CONNECTIONS_MAX: usize = 1;

/// 最大 L2CAP 通道数。
const L2CAP_CHANNELS_MAX: usize = 1;
```

## GATT 服务

我们将使用 gatt_server 和 gatt_service 宏来定义一个 GATT 服务器，其中包含一个传感器服务，该服务有两个特征。

```rust
// GATT 服务器定义
#[gatt_server]
struct Server {
    sensor_service: SensorService,
}

/// 电池服务
#[gatt_service(uuid = "a9c81b72-0f7a-4c59-b0a8-425e3bcf0a0e")]
struct SensorService {
    #[characteristic(uuid = "13c0ef83-09bd-4767-97cb-ee46224ae6db", read, notify)]
    sensor_data: u8,

    #[characteristic(uuid = "c79b2ca7-f39d-4060-8168-816fa26737b7", write, read)]
    sensor_settings: bool,
}
```

Server 结构体代表我们的 GATT 服务器，包含一个 SensorService。SensorService 定义了两个特征：

- sensor_data：一个 "u8" 值，在本练习中模拟传感器数据。它支持读取和通知（notify）操作，允许客户端读取当前传感器值并在其变化时接收通知。稍后我们将在循环中更新此值以演示实时更新。一旦你学会了如何使用 BLE，你也可以尝试将其与真实传感器一起使用，并向你的移动设备发送数据。

- sensor_settings：一个布尔值，支持写入和读取操作，使客户端能够配置传感器设置并读取当前配置。这是为了展示我们可以从客户端修改值。

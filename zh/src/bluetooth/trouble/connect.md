# 使用 nRF Connect 移动应用连接我们的 BLE 服务器

将代码烧录（flash）到 ESP32 后，打开 nRF Connect 移动应用。

扫描我们设置的蓝牙名称（我的是 "implRust"）并连接到它。

<img style="display: block; margin: auto;" alt="BLE nRF Connect mobile" src="./images/ble-scan-and-connect-with-nrf-connect.jpg"/>

应用将显示支持的服务和特征。点击特征下方的向下箭头读取数据，点击向上箭头写入数据。如果你发送（即写入）数据，你将在系统控制台中看到它。

<img style="display: block; margin: auto;" alt="BLE Services and characteristics" src="./images/ble-services-characteristics.jpg"/>

点击特征下方带有三个向下箭头的图标以订阅通知并观察值的变化。

<img style="display: block; margin: auto;" alt="BLE subscribe to notification" src="./images/ble-subscribe-to-notification.jpg"/>

## 克隆现有项目

你可以克隆（或参考）我创建的项目，并导航到 `bluetooth-low-energy` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/bluetooth-low-energy/
```

## ble 模块的完整代码

```rust
use defmt::{info, warn};

use embassy_futures::join::join;
use embassy_futures::select::select;

use embassy_time::Timer;

// BLE:
use trouble_host::prelude::*;

/// 最大连接数
const CONNECTIONS_MAX: usize = 1;

/// 最大 L2CAP 通道数。
const L2CAP_CHANNELS_MAX: usize = 1;

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

/// 运行 BLE 协议栈。
pub async fn run<C>(controller: C)
where
    C: Controller,
{
    // 使用固定的"随机"地址对测试很有用。在实际场景中，可以使用
    // 例如 MAC 6 字节数组作为地址（如何获取因平台而异）。
    let address: Address = Address::random([0xff, 0x8f, 0x1a, 0x05, 0xe4, 0xff]);
    info!("Our address = {:?}", address);

    let mut resources: HostResources<DefaultPacketPool, CONNECTIONS_MAX, L2CAP_CHANNELS_MAX> =
        HostResources::new();
    let stack = trouble_host::new(controller, &mut resources).set_random_address(address);
    let Host {
        mut peripheral,
        runner,
        ..
    } = stack.build();

    info!("Starting advertising and GATT service");
    let server = Server::new_with_config(GapConfig::Peripheral(PeripheralConfig {
        name: "implRust",
        appearance: &appearance::power_device::GENERIC_POWER_DEVICE,
    }))
    .unwrap();

    let _ = join(ble_task(runner), async {
        loop {
            match advertise("impl Rust", &mut peripheral, &server).await {
                Ok(conn) => {
                    let a = gatt_events_task(&server, &conn);
                    let b = custom_task(&server, &conn, &stack);
                    select(a, b).await;
                }
                Err(e) => {
                    let e = defmt::Debug2Format(&e);
                    panic!("[adv] error: {:?}", e);
                }
            }
        }
    })
    .await;
}

/// 这是一个需要在后台与其他 BLE 任务一起永远运行的后台任务。
///
/// ## 替代方案
///
/// 如果你的应用不需要它是通用的，你可以静态生成它，例如
///
/// ```rust,ignore
///
/// #[embassy_executor::task]
/// async fn ble_task(mut runner: Runner<'static, SoftdeviceController<'static>>) {
///     runner.run().await;
/// }
///
/// spawner.must_spawn(ble_task(runner));
/// ```
async fn ble_task<C: Controller, P: PacketPool>(mut runner: Runner<'_, C, P>) {
    loop {
        if let Err(e) = runner.run().await {
            let e = defmt::Debug2Format(&e);
            panic!("[ble_task] error: {:?}", e);
        }
    }
}

/// 流式传输事件直到连接关闭。
///
/// 此函数将处理 GATT 事件并处理它们。
/// 这是我们与读取和写入请求交互的方式。
async fn gatt_events_task<P: PacketPool>(
    server: &Server<'_>,
    conn: &GattConnection<'_, '_, P>,
) -> Result<(), Error> {
    let sensor_data = server.sensor_service.sensor_data;
    let reason = loop {
        match conn.next().await {
            GattConnectionEvent::Disconnected { reason } => break reason,
            GattConnectionEvent::Gatt { event } => {
                match &event {
                    GattEvent::Read(event) => {
                        if event.handle() == sensor_data.handle {
                            let value = server.get(&sensor_data);
                            info!(
                                "[gatt] Read Event to Sensor Data Characteristic: {:?}",
                                value
                            );
                        }
                    }
                    GattEvent::Write(event) => {
                        if event.handle() == sensor_data.handle {
                            info!(
                                "[gatt] Write Event to Sensor Data Characteristic: {:?}",
                                event.data()
                            );
                        }
                    }
                    _ => {}
                };
                match event.accept() {
                    Ok(reply) => reply.send().await,
                    Err(e) => warn!("[gatt] error sending response: {:?}", e),
                };
            }
            _ => {} // 忽略其他 Gatt 连接事件
        }
    };
    info!("[gatt] disconnected: {:?}", reason);
    Ok(())
}

/// 创建一个广播器以连接到 BLE 中心设备，并等待它连接。
async fn advertise<'values, 'server, C: Controller>(
    name: &'values str,
    peripheral: &mut Peripheral<'values, C, DefaultPacketPool>,
    server: &'server Server<'values>,
) -> Result<GattConnection<'values, 'server, DefaultPacketPool>, BleHostError<C::Error>> {
    let mut advertiser_data = [0; 31];
    let len = AdStructure::encode_slice(
        &[
            AdStructure::Flags(LE_GENERAL_DISCOVERABLE | BR_EDR_NOT_SUPPORTED),
            AdStructure::CompleteLocalName(name.as_bytes()),
        ],
        &mut advertiser_data[..],
    )?;
    let advertiser = peripheral
        .advertise(
            &Default::default(),
            Advertisement::ConnectableScannableUndirected {
                adv_data: &advertiser_data[..len],
                scan_data: &[],
            },
        )
        .await?;
    info!("[adv] advertising");
    let conn = advertiser.accept().await?.with_attribute_server(server)?;
    info!("[adv] connection established");
    Ok(conn)
}

/// 使用 BLE 通知器接口的示例任务。
/// 此任务将每 2 秒向连接的中心设备通知一个计数器值。
/// 它还将每 2 秒读取一次 RSSI 值。
/// 当中心设备关闭连接或发生错误时，它将停止。
async fn custom_task<C: Controller, P: PacketPool>(
    server: &Server<'_>,
    conn: &GattConnection<'_, '_, P>,
    stack: &Stack<'_, C, P>,
) {
    let mut tick: u8 = 0;
    let sensor_data = server.sensor_service.sensor_data;
    loop {
        tick = tick.wrapping_add(1);
        info!("[custom_task] notifying connection of tick {}", tick);
        if sensor_data.notify(conn, &tick).await.is_err() {
            info!("[custom_task] error notifying connection");
            break;
        };
        if let Ok(rssi) = conn.raw().rssi(stack).await {
            info!("[custom_task] RSSI: {:?}", rssi);
        } else {
            info!("[custom_task] error getting RSSI");
            break;
        };
        Timer::after_secs(2).await;
    }
}
```

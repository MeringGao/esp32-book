# BLE 广播

BLE 广播是设备向附近设备（如手机或电脑）宣布其存在并共享信息以便它们可以连接的方式。它广播设备名称、可用服务和功能。

一旦中心设备发起连接，该函数就接受它并返回一个 GATT 连接实例，可用于与连接的设备通信。

```rust
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
```

让我们分解这个函数并理解每一步。

## 准备广播数据

```rust
let mut advertiser_data = [0; 31];
 
let len = AdStructure::encode_slice(
    &[
        AdStructure::Flags(LE_GENERAL_DISCOVERABLE | BR_EDR_NOT_SUPPORTED),
        AdStructure::CompleteLocalName(name.as_bytes()),
    ],
    &mut advertiser_data[..],
)?;
```

我们创建一个缓冲区并编码两条信息：

- 标志（Flags）：`LE_GENERAL_DISCOVERABLE` 使设备对所有扫描器可见，而 `BR_EDR_NOT_SUPPORTED` 表示这是一个仅 BLE 设备（不支持经典蓝牙）。你可以在[这里](../ble/gap.html#广播标志)找到有关这些标志的更多详细信息。
- 设备名称（Device Name）：扫描设备时显示的完整本地名称

`encode_slice` 函数将这些数据打包成正确的 BLE 广播格式，并返回实际使用的长度。

## 开始广播

```rust
let advertiser = peripheral
    .advertise(
        &Default::default(),
        Advertisement::ConnectableScannableUndirected {
            adv_data: &advertiser_data[..len],
            scan_data: &[],
        },
    )
    .await?;
```

我们使用 `ConnectableScannableUndirected` 模式开始广播，这意味着：
- 可连接（Connectable）：中心设备可以连接到我们
- 可扫描（Scannable）：中心设备可以请求额外信息（尽管我们在这里提供空的 `scan_data`）
- 无定向（Undirected）：我们向所有人广播，而不是针对特定设备

该设备现在对附近的 BLE 扫描器可见。

## 接受连接

```rust
let conn = advertiser.accept().await?.with_attribute_server(server)?;
Ok(conn)
```

`accept()` 调用等待中心设备发起连接。连接后，我们使用 `with_attribute_server()` 将连接转换为 `GattConnection`，这将我们的 GATT 服务器（包含我们的服务和特征）附加到连接上。

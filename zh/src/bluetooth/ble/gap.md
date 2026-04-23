# 通用访问配置文件（GAP）

GAP（Generic Access Profile，通用访问配置文件）是一组规则，控制蓝牙低功耗（BLE）设备如何发现、连接和相互通信。

## BLE 通信类型

BLE 支持两种主要的通信方式：**连接通信（connected communication）**和**广播通信（broadcast communication）**。

**连接通信：** 两个设备形成直接连接，允许它们双向发送和接收数据。例如，智能手表连接到手机并持续共享心率、通知和步数等数据。

**广播通信：** 设备向所有附近设备发送数据，而不建立直接连接。例如，商店中的蓝牙信标（beacon）向范围内所有手机广播促销信息。

## 设备角色

想象这些角色就像现实世界中的人类交流。就像人们根据在对话中的角色以不同方式互动一样，蓝牙低功耗（BLE）设备也有特定的角色。

**📢 广播者（Broadcaster，无连接）**：发送信息（广播数据包），但无法被连接。
例如，商场中的信标持续向附近智能手机发送折扣优惠。手机可以接收优惠，但无法连接到信标。

**📡 观察者（Observer，无连接）**：监听蓝牙广播，但无法连接到其他设备。
例如，智能手机应用扫描信标以检测附近商店，但不连接它们。

**📱 中心设备（Central，面向连接）**：该设备搜索其他设备，连接到它们，或读取它们的广播数据。它通常具有更强的处理能力和资源。它可以同时处理多个连接。

例如，智能手机同时连接智能手表、健身追踪器和无线耳机。

**⌚ 外设（Peripheral，面向连接）**：该设备广播广播数据包并接受来自中心设备的连接请求。
例如，健身追踪器广播自身，以便智能手机可以发现并连接它以同步健康数据。

<img style="display: block; margin: auto;" alt="Central And Peripherals" src="../images/ble-central-peripheral.jpg"/>

## BLE 外设发现模式与广播标志

BLE 外设可以处于不同的发现模式，影响中心设备如何检测到它。这些模式使用广播数据包中的广播标志（advertisement flags）设置。

### 发现模式

1. **不可发现（Non-Discoverable）**
   - 未激活广播或建立连接时的默认模式。
   - 无法被发现或连接。

2. **有限可发现（Limited-Discoverable）**
   - 在**有限时间内**可发现以节省电量。
   - 如果未建立连接，设备将进入空闲状态。

3. **一般可发现（General-Discoverable）**
   - 持续广播，直到建立连接。

### 广播标志

这些标志指示发现模式和 BLE 支持级别。它们使用按位或（`|`）组合：

| 位 | 标志（在 [`TrouBLE`](https://github.com/embassy-rs/trouble) crate 中） | 描述 |
|----|--------------------------------|------------------------------------------------|
| 0 | `AD_FLAG_LE_LIMITED_DISCOVERABLE` | 有限可发现模式（临时广播）。 |
| 1 | `LE_GENERAL_DISCOVERABLE` | 一般可发现模式（无限期广播）。 |
| 2 | `BR_EDR_NOT_SUPPORTED` | 当设备**不支持**（或不想支持）蓝牙经典（BR/EDR）时设置。 |
| 3 | `SIMUL_LE_BR_CONTROLLER` | 如果设备可以同时使用蓝牙低功耗（LE）和经典蓝牙（控制器级别），则设置。|
| 4 | `SIMUL_LE_BR_HOST` | 如果设备可以同时运行蓝牙低功耗（LE）和经典蓝牙（主机级别），则设置。 |
| 5-7 | 保留 | 未使用。 |

使用 Bleps crate 的示例：

```rust
create_advertising_data(&[
   // 标志
   AdStructure::Flags(LE_GENERAL_DISCOVERABLE | BR_EDR_NOT_SUPPORTED)
   // 其他广播数据
]).unwrap()
```

使用 Trouble crate 的示例：

```rust
    let mut adv_data = [0; 31];
    let len = AdStructure::encode_slice(
        &[
            // 标志
            AdStructure::Flags(LE_GENERAL_DISCOVERABLE | BR_EDR_NOT_SUPPORTED),
            // 其他广播数据
        ],
        &mut adv_data[..],
    )
    .unwrap();
```

这将外设配置为无限期广播（LE_GENERAL_DISCOVERABLE），同时指示它不支持蓝牙经典（BR_EDR_NOT_SUPPORTED）。

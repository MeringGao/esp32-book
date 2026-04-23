# 运行 BLE 协议栈

在这里，我们定义 `run` 函数，该函数在 ESP32 上设置并启动 BLE 协议栈。

## 整体流程

<img style="display: block; margin: auto;" alt="BLE Overall flow of the code" src="./images/overall-flow.svg"/>

```rust
/// 运行 BLE 协议栈。
pub async fn run<C>(controller: C)
where
    C: Controller,
{
    // 使用固定的"随机"地址对测试很有用。在实际场景中，可以使用
    // 例如 MAC 6 字节数组作为地址（如何获取因平台而异）。
    // Using a fixed "random" address can be useful for testing. In real scenarios, one would
    // use e.g. the MAC 6 byte array as the address (how to get that varies by the platform).
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
                    // 当与中心设备建立连接时设置任务，这样没有连接时它们不会运行。
                    // set up tasks when the connection is established to a central, so they don't run when no one is connected.
                    let a = gatt_events_task(&server, &conn);
                    let b = custom_task(&server, &conn, &stack);
                    // 运行直到任一任务结束（通常是因为连接已关闭），
                    // 然后返回广播状态。
                    // run until any task ends (usually because the connection has been closed),
                    // then return to advertising state.
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
```

让我们分解这个函数并理解每一步。该函数接受一个名为 "controller" 的参数，它必须实现 Trouble crate 提供的 `Controller` trait。

## 设置 BLE 主机

让我们创建 BLE 主机实例。为此，我们首先需要定义 BLE 地址。

### BLE 地址

每个 BLE 设备都有一个唯一的蓝牙设备地址，这是一个 48 位数字，类似于 MAC 地址。

<img style="display: block; margin: auto;" alt="BLE Address Categories" src="./images/ble-address-type.svg"/>

BLE 地址有两种主要类型：

**公共地址（Public Address）**：公共地址是由制造商分配给设备的永久性、全球唯一代码。它永远不会改变，并在 IEEE 注册。每个地址只能由一个设备拥有，获取一个需要支付费用。

**随机地址（Random Address）**：随机地址更常用，因为你不需要在 IEEE 注册，而且你可以自己设置。

随机地址可以进一步分类为

- *静态（Static）*：直到你重启设备前保持不变
- *私有（动态）（Private (dynamic)）*：随时间变化以保护隐私。可以是可追踪的（带有身份解析密钥）或完全匿名的。随机地址通过隐藏设备的真实身份来帮助保护隐私。

```rust
// 使用固定的"随机"地址对测试很有用。在实际场景中，可以使用
// 例如 MAC 6 字节数组作为地址（如何获取因平台而异）。
// Using a fixed "random" address can be useful for testing. In real scenarios, one would
// use e.g. the MAC 6 byte array as the address (how to get that varies by the platform).
let address: Address = Address::random([0xff, 0x8f, 0x1a, 0x05, 0xe4, 0xff]);
info!("Our address = {:?}", address);
```

### 初始化 BLE 主机

让我们通过提供控制器和资源配置来初始化 BLE 主机。然后，我们设置主机将使用的随机地址。最后，我们在协议栈上调用 build 方法，它为我们提供 BLE 外设（Peripheral）和运行器（runner）。

```rust

let mut resources: HostResources<DefaultPacketPool, CONNECTIONS_MAX, L2CAP_CHANNELS_MAX> =
    HostResources::new();
let stack = trouble_host::new(controller, &mut resources).set_random_address(address);
let Host {
    mut peripheral,
    runner,
    ..
} = stack.build();
```

### 初始化 GATT 服务器

`gatt_server` 宏为 `Server` 结构体初始化各种组件，包括对 builder 方法 new_with_config 的支持。

在这个示例中，我们将设备配置为作为[外设（Peripheral）](../ble/gap.html#设备角色)运行，并为其分配名称 "implRust"。你可以将其更改为任何你喜欢的名称。

```rust
let server = Server::new_with_config(GapConfig::Peripheral(PeripheralConfig {
    name: "implRust",
    appearance: &appearance::power_device::GENERIC_POWER_DEVICE,
}))
.unwrap();
```

## 任务

服务器配置完成后，我们需要运行两个并发任务：BLE 协议栈任务和我们的应用逻辑。join 方法允许两个任务同时运行。

```rust
let _ = join(ble_task(runner), async {
    loop {
        match advertise("impl Rust", &mut peripheral, &server).await {
            Ok(conn) => {
                // 当与中心设备建立连接时设置任务，这样没有连接时它们不会运行。
                // set up tasks when the connection is established to a central, so they don't run when no one is connected.
                let a = gatt_events_task(&server, &conn);
                let b = custom_task(&server, &conn, &stack);
                // 运行直到任一任务结束（通常是因为连接已关闭），
                // 然后返回广播状态。
                // run until any task ends (usually because the connection has been closed),
                // then return to advertising state.
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
```

我们很快就会定义并查看 `advertise` 函数。该函数返回一个 `Result`，其中包含 GATT 连接或 BLE 主机错误。如果我们成功获得连接，我们将生成两个连接特定任务：

- gatt_events_task：处理来自连接的中心设备的传入 GATT 请求（读取、写入）
- custom_task：在连接期间运行我们的应用特定逻辑

`select` 函数同时运行两个任务，并在任一任务完成时完成；通常在连接丢失时发生。一旦这种情况发生，循环将返回广播状态，使设备再次对新连接可发现。

## BLE 协议栈任务

此任务在后台与其他 BLE 任务一起持续运行，以保持蓝牙协议栈处于活动状态。

```rust
async fn ble_task<C: Controller, P: PacketPool>(mut runner: Runner<'_, C, P>) {
    loop {
        if let Err(e) = runner.run().await {
            let e = defmt::Debug2Format(&e);
            panic!("[ble_task] error: {:?}", e);
        }
    }
}
```

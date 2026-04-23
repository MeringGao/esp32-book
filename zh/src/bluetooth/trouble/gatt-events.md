# GATT 事件

GATT 事件任务处理来自连接的中心设备的传入请求。它处理特征的读取和写入操作，并适当响应，直到连接关闭。

```rust
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
                // 此步骤也在 drop() 中执行，但显式编写是必要的
                // 以确保回复被发送。
                // This step is also performed at drop(), but writing it explicitly is necessary
                // in order to ensure reply is sent.
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
```

让我们分解这个函数并理解每一步。

## 事件循环

```rust
let sensor_data = server.sensor_service.sensor_data;

let reason = loop {
    match conn.next().await {
        GattConnectionEvent::Disconnected { reason } => break reason,
        GattConnectionEvent::Gatt { event } => {
            // ... 处理事件
        }
        _ => {} // 忽略其他 Gatt 连接事件
    }
};
info!("[gatt] disconnected: {:?}", reason);
```

我们首先获取对传感器数据特征的引用，然后进入一个等待连接上事件的循环。循环继续直到发生断开连接事件，此时它跳出并记录断开连接原因。

## 处理读取事件

```rust
GattEvent::Read(event) => {
    if event.handle() == sensor_data.handle {
        let value = server.get(&sensor_data);
        info!(
            "[gatt] Read Event to Sensor Data Characteristic: {:?}",
            value
        );
    }
}
```

当中心设备读取特征时，我们检查句柄是否与我们的传感器数据特征匹配。如果是，我们从服务器检索当前值并记录它。当调用 accept 和 send 时，库会自动处理发送值。

## 处理写入事件

```rust
GattEvent::Write(event) => {
    if event.handle() == sensor_data.handle {
        info!(
            "[gatt] Write Event to Sensor Data Characteristic: {:?}",
            event.data()
        );
    }
}
```

当中心设备写入特征时，我们检查它是否写入到我们的传感器数据特征。如果是，我们记录正在写入的数据。当我们调用 `accept()` 时，实际写入将被处理。

## 发送响应

```rust
match event.accept() {
    Ok(reply) => reply.send().await,
    Err(e) => warn!("[gatt] error sending response: {:?}", e),
};
```

处理每个事件后，我们必须显式接受它并向客户端发送响应。虽然当事件被丢弃时这也会自动发生，但显式执行可以确保响应立即发送。如果发送响应失败，我们记录警告。

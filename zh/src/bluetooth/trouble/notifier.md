# 通知器（Notifier）

此任务演示如何向连接的中心设备推送数据。想象它就像你定期收集并发送到移动设备或其他设备的传感器数据。在这个示例中，我们通过每 2 秒递增一个计数器来模拟传感器读数，并向连接的设备发送通知。

```rust
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
        // 读取连接的 RSSI（接收信号强度指示器）。
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

## 发送通知

```rust
let mut tick: u8 = 0;
let sensor_data = server.sensor_service.sensor_data;
loop {
    tick = tick.wrapping_add(1);
    info!("[custom_task] notifying connection of tick {}", tick);
    if sensor_data.notify(conn, &tick).await.is_err() {
        info!("[custom_task] error notifying connection");
        break;
    };
    // ...
    Timer::after_secs(2).await;
}
```

我们维护一个计数器，每次循环迭代递增。`notify()` 方法将当前计数器值发送给连接的中心设备。如果通知失败（通常是因为连接断开），我们退出循环。

## 读取信号强度

```rust
if let Ok(rssi) = conn.raw().rssi(stack).await {
    info!("[custom_task] RSSI: {:?}", rssi);
} else {
    info!("[custom_task] error getting RSSI");
    break;
};
```

RSSI（Received Signal Strength Indicator，接收信号强度指示器）以 dBm 为单位测量连接质量。我们读取此值以监控连接强度。如果读取 RSSI 失败，通常表示连接已丢失，因此我们退出任务。

# ESP32 的 I2C 总线接口

ESP32 包含两个 I2C 总线接口，每个接口都可以配置为控制器（controller）或目标设备（target）。根据 ESP32 的数据手册（datasheet），一个接口属于主系统，另一个则属于低功耗子系统。

两个接口都支持标准模式（100 Kbit/s）、快速模式（400 Kbit/s），最高可达 5 MHz。不过实际性能取决于 SDA 和 SCL 线上拉电阻（pull-up resistor）的强度。ESP32 支持 7 位和 10 位寻址，包括双地址模式。

得益于灵活的 GPIO 矩阵，SDA 和 SCL 信号可以分配到几乎任意 GPIO 引脚（pin）上。

## 在 esp-hal 中使用 I2C

要在 esp-hal crate 中使用 I2C，我们需要手动配置接口并分配引脚。以下示例展示了如何将 GPIO18 配置为 SCL、GPIO23 配置为 SDA 来设置 I2C：

```rust
let i2c_bus = esp_hal::i2c::master::I2c::new(
    peripherals.I2C0,
    esp_hal::i2c::master::Config::default().with_frequency(Rate::from_khz(400)),
)
.unwrap()
.with_scl(peripherals.GPIO18)
.with_sda(peripherals.GPIO23);
```

总线准备就绪后，我们可以将其传递给驱动，例如 OLED 显示屏的驱动：

```rust
let interface = I2CDisplayInterface::new(i2c_bus);
// 初始化显示屏
let mut display = Ssd1306Async::new(interface, DisplaySize128x64, DisplayRotation::Rotate0)
    .into_buffered_graphics_mode();
```

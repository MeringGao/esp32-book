# 在嵌入式 Rust 生态中使用 I2C

在上一节中，我们学习了 I2C 通信的基础知识以及控制器-目标设备（controller-target，旧称 master-slave）模型的工作原理。现在，让我们看看这些概念如何应用于嵌入式 Rust 生态——其中模块化和可复用设计是一项核心原则。

## embedded-hal 的作用

embedded-hal crate 定义了一套标准的嵌入式硬件抽象 trait，其中就包括 I2C。这些 trait 使得驱动代码（例如显示屏或传感器的驱动）可以以通用的方式编写，从而无需针对特定平台进行修改即可在多种不同的微控制器上运行。

核心的 I2C trait 如下所示：

```rust
pub trait I2c<A: AddressMode = SevenBitAddress>: ErrorType {
    // HAL 作者必须实现这个方法
    fn transaction(...);
    // 以下是以 transaction 为基础提供的默认方法
    fn read(...);
    fn write(...);
    fn write_read(...);
}
```

HAL 唯一需要实现的方法是 `transaction`。该 trait 基于 `transaction` 提供了 `read`、`write` 和 `write_read` 的默认实现。

泛型参数 `A` 用于指定地址模式（address mode），默认类型参数为 `SevenBitAddress`。因此在大多数情况下你无需手动指定它。如果需要使用 10 位地址，可以使用 `TenBitAddress`。

各微控制器专用的 HAL crate（例如 esp-hal、stm32-hal 或 nrf-hal）会为各自的 I2C 外设（peripheral）实现这个 trait。例如，esp-hal crate 就实现了 I2C trait。如果你感兴趣，可以在[这里](https://github.com/esp-rs/esp-hal/blob/de67c3101346cdbe030ffa1bb95b13943ee8d790/esp-hal/src/i2c/master/mod.rs#L671)查看其实现。

> 除了常规的 embedded-hal crate 之外，还有一个异步版本叫做 embedded-hal-async。它定义了类似的 trait，但专为异步（async）代码设计，在编写非阻塞驱动或嵌入式系统中的异步任务时非常有用。

## 平台无关的驱动

假设你正在编写一个通过 I2C 通信的传感器或显示屏驱动。你不希望代码被绑定到特定的微控制器（例如 Raspberry Pi Pico 或 ESP32）。相反，你可以使用 embedded-hal trait 以通用的方式编写驱动。

只要你的驱动仅依赖 I2C trait，它就可以在任何提供了该 trait 实现的平台上运行——例如 STM32、nRF 或 ESP32。

## 共享 I2C 总线

许多嵌入式项目将多个 I2C 设备（例如 OLED 显示屏、LCD 和各种传感器）连接到同一组 SDA 和 SCL 线上。然而，同一时间只能有一个设备控制总线。

<img style="display: block; margin: auto;" alt="I2C 单控制器与多设备" src="./images/ic2-mcu-multi-device.svg"/>
<p align="center"><em>图：微控制器（ESP32）与多个设备</em></p>

如果你将总线的独占访问权交给某一个驱动，其他设备就无法通信。这时 embedded-hal-bus crate 就派上用场了。

它提供了诸如 `AtomicDevice`、`CriticalSectionDevice` 和 `RefCellDevice` 之类的包装类型，允许多个驱动安全地共享对同一条 I2C 总线的访问。这些包装器本身也实现了 I2c trait，因此驱动可以像使用原始总线一样使用它们。

你可以通过两种方式使用 I2C：

<img style="display: block; margin: auto;" alt="I2C 单控制器多设备" src="./images/i2c-embedded-hal-rust-ecosystem.svg"/>

- **不共享**：如果你的应用只与一个 I2C 设备通信，可以直接将 HAL 提供的 I2C 总线实例（实现了 I2c trait）传递给驱动。

- **共享**：如果你的应用需要在同一总线上与多个 I2C 设备通信，可以使用 embedded-hal-bus crate 中的共享类型（例如 `AtomicDevice` 或 `CriticalSectionDevice`）包装 HAL 提供的 I2C 总线实例。这样可以实现多个驱动之间的安全、协调访问。

## 参考资料

- [embedded-hal docs on I2C](https://docs.rs/embedded-hal/latest/embedded_hal/i2c/index.html)：该文档详细介绍了 I2C trait 的结构以及它们在不同平台上的使用方式。

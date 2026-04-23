# 集成电路互联总线（Inter-Integrated Circuit，I2C）

在本节中，我们将学习什么是 I2C，以及如何使用 ESP32 的 I2C 通信总线。

## 什么是 I2C？

I2C 全称 Inter-Integrated Circuit，也写作 I²C。它是微控制器（microcontroller）与传感器、显示屏（例如 OLED）以及其他芯片通信时常用的一种通信方式。I2C 是一种串行（serial）、半双工（half-duplex）、同步（synchronous）接口。我们来逐一解释这些术语的含义。

- **串行（serial）** 意味着数据通过单条数据线逐位传输。可以想象成一座单车道桥梁，车辆（数据位）依次排队通过。

- **半双工（half-duplex）** 意味着数据同一时间只能朝一个方向传输。就像使用对讲机——一方说话时另一方只能收听，然后双方再交换角色。

- **同步（synchronous）** 意味着两个设备依赖共享的时钟信号来协调通信。想象两个人互相传球，但只有在裁判吹哨时才传。哨声就像时钟信号，确保双方保持同步。

## 控制器与目标设备

I2C 采用控制器-目标设备（controller-target）模型。控制器（controller，旧称 master）负责发起通信并提供时钟信号；目标设备（target，旧称 slave）则响应控制器的指令。

<img style="display: block; margin: auto;" alt="I2C 单控制器与单目标设备" src="./images/i2c-bus.svg"/>
<p align="center"><em>图：单控制器与单目标设备</em></p>

在典型的嵌入式项目中，微控制器（例如 ESP32）充当控制器，而连接的显示屏（例如 OLED）或传感器则充当目标设备。

I2C 的优势在于只需两根线即可连接多个设备。你可以将多个目标设备连接到同一个控制器上，这是最常用的配置。I2C 也支持同一总线上存在多个控制器，因此多个控制器可以与一个或多个目标设备通信。

## I2C 总线

I2C 总线仅使用两条线路，所有连接的设备共享这两条线：

- **SCL（Serial Clock Line，串行时钟线）**：由控制器提供时钟信号。有些设备会将其标注为 SCK。

- **SDA（Serial Data Line，串行数据线）**：双向传输数据。有些设备会将其标注为 SDI。

<img style="display: block; margin: auto;" alt="I2C 单控制器与多目标设备" src="./images/ic2-multi-target-single-controller.svg"/>
<p align="center"><em>图：单控制器与多目标设备</em></p>

所有连接的设备共享相同的两根线。控制器通过发送目标设备的唯一地址（address）来选择与哪个设备通信。

## I2C 地址

每个 I2C 目标设备都有一个 7 位或 10 位的地址。最常见的是 7 位地址，最多支持 128 个不同地址。

许多设备的地址由制造商固定，但也有一些允许通过引脚或跳线配置地址的低位。例如，某个传感器可能使用标有 A0 和 A1 的引脚来改变地址，从而允许你在同一总线上使用多个相同的芯片。

当控制器想要与某个目标设备通信时，它会先发送一个起始条件（start condition），然后是设备地址和一个读/写位。匹配的设备会回复一个 ACK（acknowledge，应答）信号，随后通信继续进行。

## 速度模式

I2C 支持不同的速度模式（speed mode），具体取决于数据传输速率的需求：

- 标准模式（Standard mode）：最高 100 kbps
- 快速模式（Fast mode）：最高 400 kbps
- 快速模式增强（Fast Mode Plus）：最高 1 Mbps
- 高速模式（High-Speed mode）：最高 3.4 Mbps
- 超快速模式（Ultra-Fast mode）：最高 5 Mbps

实际可使用的速度取决于微控制器的 I2C 接口和连接的目标设备所支持的速度模式。

## 为什么选择 I2C？

当你希望用尽可能少的连线连接多个设备时，I2C 是理想选择。它非常适合对速度要求不高、但布线简洁性更重要的应用场景。

好消息是，在嵌入式 Rust 中，你无需自己实现 I2C 协议。embedded-hal crate 定义了通用的 I2C trait，而你所使用芯片的 HAL 则负责处理底层细节。下一节我们将详细介绍。

## 参考资料

- [Basics of the I2C Communication Protocol](https://www.circuitbasics.com/basics-of-the-i2c-communication-protocol/)：如果你想深入了解控制器如何与目标设备通信，可以参考这篇文章。

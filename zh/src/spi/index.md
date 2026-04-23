# 串行外设接口（Serial Peripheral Interface，SPI）

在本节中，我们将学习什么是 SPI，以及如何使用 ESP32 的 SPI 通信总线。

## 什么是 SPI？

SPI 全称 Serial Peripheral Interface（串行外设接口）。它是微控制器与显示屏、传感器和 SD 卡等设备通信的最常见方式之一。从技术上讲，它是一种串行（serial）、全双工（full duplex）、同步（synchronous）接口。但这些术语是什么意思呢？

- **串行（serial）** 意味着数据逐位发送。想象一座单车道桥梁，一次只能有一辆车通过。每一位数据就像一辆汽车依次排队过桥。这与多车道高速公路（并行通信）不同，后者允许多辆汽车并排同时行驶。

- **全双工（full duplex）** 意味着两个设备可以同时互相通信。想象两个人在打电话，他们可以同时说话和收听。这就是全双工。相比之下，半双工（half duplex）就像使用对讲机——一方说话时另一方只能收听，然后双方交换角色。

- **同步（synchronous）** 意味着两个设备使用同一个时钟来保持同步。想象两个人在玩抛接球，但他们只在哨声响起时才抛出和接住。哨声就像时钟信号，确保双方确切知道何时发送和接收数据。

## 控制器与外设

SPI 采用控制器-外设模型（controller-peripheral model，旧称 master-slave）。控制器（controller）是发起通信并提供时钟信号的设备。外设（peripheral）是响应控制器的设备。

在大多数嵌入式项目中，ESP32 充当控制器，而显示屏、传感器或 SD 卡等设备则是外设。

<img style="display: block; margin: auto;" alt="SPI 单总线单外设" src="./images/spi-bus.svg"/>
<p align="center"><em>图：单 SPI 总线上的控制器与单个外设</em></p>

连接通常使用四条线：

- **SCK（Serial Clock，串行时钟）** 线传输由控制器生成的时钟信号。该信号在数据传输期间保持控制器和设备同步。

- **MOSI（Master Out, Slave In，主出从入）** 线用于将数据从控制器发送到设备。在某些数据手册中，这条线可能标注为 SDO（Serial Data Out，串行数据输出）。

- **MISO（Master In, Slave Out，主入从出）** 线将数据从设备传输到控制器。根据设备不同，它也可能被称为 SDI（Serial Data In，串行数据输入）。

- 最后，**CS（Chip Select，片选）** 线由控制器使用，用于选择它想要与哪个设备通信。每个连接的设备通常有自己专用的 CS 线。当控制器将某个设备的 CS 线拉低（低电平有效，active low）时，该设备变为活动状态并准备好通信。在一些较早的文档中，这条线可能被称为 SS（Slave Select，从设备选择）。

<img style="display: block; margin: auto;" alt="SPI 单总线多外设" src="./images/generic-spi-single-bus-multiple-spi-device.svg"/>

<p align="center"><em>图：单 SPI 总线上的控制器与多个外设</em></p>

### SPI 模式

SPI 支持四种模式，编号从 0 到 3。这些模式根据时钟的空闲电平（idle level）和边沿（edge）决定何时读取和写入数据。

现在不必过于担心细节。模式 0 是最常见的，适用于大多数设备。

## 为什么选择 SPI？

当你希望微控制器与外设之间进行快速可靠的通信时，SPI 是一个很好的选择。它比 I²C 或 UART 快得多，使用简单，并且允许同时发送和接收数据（全双工）。这使其成为显示屏或 SD 卡等高速设备的理想选择。

只要你有足够的 GPIO 引脚并且不需要连接大量设备，SPI 通常就是完成这项工作的最佳工具。

## 参考资料

如需更深入的技术细节，请参阅以下内容：
- [Serial Peripheral Interface (SPI) by Sparkfun](https://learn.sparkfun.com/tutorials/serial-peripheral-interface-spi/all)
- [Basics of the SPI Communication Protocol](https://www.circuitbasics.com/basics-of-the-spi-communication-protocol/)

# 在嵌入式 Rust 生态中使用 SPI

在上一节中，我们学习了什么是 SPI 以及控制器-外设（controller-peripheral）模型的工作原理。现在，让我们看看这些概念如何应用于嵌入式 Rust 生态系统。

Rust 的嵌入式生态系统被设计为模块化和可复用的。这意味着你可以为一款微控制器编写代码，并在另一款上进行复用，只需做少量修改。实现这种灵活性的关键之一是使用 embedded-hal crate 定义的 trait。

## embedded-hal 中的 SPI

embedded-hal crate 定义了用于 SPI 的标准 trait，以便驱动和库可以以通用的方式编写。对于 SPI，有两个重要的 trait：

- **SpiBus**：表示对 SPI 总线的完全控制，包括 SCK、MOSI 和 MISO 线。这必须由微控制器的 HAL crate 实现。例如，esp-hal crate 就实现了 SpiBus。如果你感兴趣，可以在[这里](https://github.com/esp-rs/esp-hal/blob/de67c3101346cdbe030ffa1bb95b13943ee8d790/esp-hal/src/spi/master.rs#L2703)查看其实现。

- **SpiDevice**：表示对单个 SPI 设备的访问，该设备可能与其他设备共享总线。它控制片选（CS）引脚，并确保设备在通信前被正确选中，通信后释放。

## 平台无关的驱动

假设你正在编写一个通过 SPI 通信的传感器或显示屏驱动。你不希望代码被绑定到特定的微控制器（例如 Raspberry Pi Pico 或 ESP32）。相反，你可以使用 embedded-hal trait 以通用的方式编写驱动。

只要你的驱动仅依赖 SpiDevice 或 SpiBus trait，它就可以在任何提供了这些 trait 实现的平台上运行——例如 STM32、nRF 或 ESP32。

## 共享 SPI 总线

在许多项目中，多个 SPI 设备共享同一条 SPI 总线。例如，你可能有一个显示屏、一张 SD 卡和一个温度传感器，它们都连接到同一组 MOSI、MISO 和 SCK 线上。唯一区分它们的是各自的片选（CS）引脚。

<img style="display: block; margin: auto;" alt="SPI 单总线多外设" src="./images/mcu-spi-single-bus-multiple-spi-device.svg"/>

<p align="center"><em>图：单 SPI 总线上的控制器与多个外设</em></p>

如果你将 SPI 总线的完全控制权交给某一个驱动（使用 SpiBus），其他驱动就无法使用它。相反，我们需要一种安全地在多个设备之间共享 SPI 总线的方式。

SpiDevice 就是为此而生的——它允许每个驱动只使用自己的 CS 引脚，同时仍然共享底层总线。

在实践中，这意味着我们需要向每个驱动传递一个实现了 SpiDevice 的结构体，而不是将完整的总线交给它们。

## 如何获取 SpiDevice？

我们说过每个驱动应该被给予 SpiDevice 实现而不是完整的 SpiBus。但我们需要自己编写这个 SpiDevice 结构体吗？

不一定。虽然可以自己实现，但通常没有必要——而且当多个设备需要协调总线访问时，这可能会很棘手。

这时 embedded-hal-bus crate 就派上用场了。它提供了现成的包装器，为你实现 SpiDevice trait。这些包装器处理总线访问、片选控制以及设备之间的可选同步。

<img style="display: block; margin: auto;" alt="SPI 单总线多外设" src="./images/spi-embedded-hal-rust-ecosystem.svg"/>

- 如果你的项目只使用一个 SPI 设备且不需要共享，你可以使用 `ExclusiveDevice` 结构体——它为单个设备提供对总线的独占访问。

- 但如果你的项目有多个 SPI 设备共享同一条总线，你可以选择一种共享访问实现，例如 `AtomicDevice` 或 `CriticalSectionDevice`。它们管理对总线的访问，使每个设备都能轮流使用而不会互相干扰。

这些结构体让你可以专注于使用或构建驱动，而无需担心底层协调或编写样板代码。

## 参考资料

- [embedded-hal docs on SPI](https://docs.rs/embedded-hal/latest/embedded_hal/spi/index.html)：该文档详细介绍了 SPI trait 的结构以及它们在不同平台上的使用方式。

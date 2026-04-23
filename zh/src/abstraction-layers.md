# 抽象层（Abstraction Layers）

在使用嵌入式 Rust 时，你经常会遇到 PAC、HAL 和 BSP 等术语。这些是帮助你与硬件交互的不同层次。每一层在灵活性和易用性之间提供了不同的平衡。

让我们从最高级别的抽象开始，一直到最低级别。

<a href ="./images/abstraction-layers.png"><img alt="abstraction layers" style="display: block; margin: auto;" src="./images/abstraction-layers.png"/></a>


## 板级支持包（BSP, Board Support Package）

BSP，在 Rust 中也被称为板级支持 Crate（Board Support Crate），针对特定的开发板进行定制。它将 HAL 与板级特定的配置相结合，为板载组件（如 LED、按钮和传感器）提供即用的接口。这使开发者能够专注于应用逻辑，而无需处理底层硬件细节。由于目前没有专门针对 ESP32 DevKit v1 的流行 BSP，我们在本书中不会使用这种方法。

---

## 硬件抽象层（HAL, Hardware Abstraction Layer）

HAL 位于 BSP 级别之下。如果你使用 Raspberry Pi Pico 或基于 ESP32 的开发板，你大部分时间都会使用 HAL 级别。

HAL 构建在 PAC 之上，为微控制器的外设（Peripheral）提供更简单、更高级的接口。HAL 提供了方法和 trait，使设置定时器、配置串行通信或控制 GPIO 引脚等任务变得更加容易，而无需直接处理底层寄存器（Register）。

微控制器的 HAL 通常实现 `embedded-hal` trait，这是用于 GPIO、SPI、I2C 和 UART 等外设的标准化、平台无关的接口。这使得编写驱动程序和库变得更加容易，只要使用兼容的 HAL，它们就可以在不同的硬件上工作。

在本书中，我们将使用 `esp-hal` crate。同一个 HAL 也可以用于其他 ESP32 变体。要切换到不同的 ESP32 系列，只需更新 `Cargo.toml` 文件中的特性标志（Feature Flag）。


---
> [!Note]
> HAL 以下的层次很少直接使用。在大多数情况下，PAC 是通过 HAL 访问的，而不是单独使用。除非你使用的芯片没有可用的 HAL，否则通常不需要直接与下层交互。在本书中，我们将专注于 HAL 层。


## 外设访问 Crate（PAC, Peripheral Access Crate）

PAC 是最低级别的抽象。它们是自动生成的 crate，提供对微控制器外设（Peripheral）的类型安全访问。这些 crate 通常使用 `svd2rust` 等工具从制造商的 SVD（System View Description）文件生成。PAC 为你提供了以结构化且安全的方式直接与硬件寄存器（Register）交互的能力。

## 微架构 Crate（Micro Architecture Crate）

这在抽象层次中与 PAC 并列。这些 crate 特定于微控制器中使用的处理器核心架构（例如 ARM Cortex 或 Xtensa）。它们提供对核心功能和共享内部外设（Peripheral）的低级别访问。

对于 ESP32，这将是 xtensa-lx 和 xtensa-lx-rt crate。这些 crate 处理启用或禁用中断（Interrupt）以及访问内部定时器等操作。对于基于 ARM Cortex 的微控制器，例如 STM32、nRF 或 RP2040 系列，等效的 crate 是 cortex-m 和 cortex-m-rt。这些微架构 crate 构成了更高级别的 HAL 和 BSP 构建的基础，确保共享相同核心的芯片之间的兼容性和可重用性。

## 原始 MMIO（Raw MMIO）

原始 MMIO（Memory-Mapped I/O，内存映射 I/O）意味着通过向特定内存地址读取和写入来直接操作硬件寄存器（Register）。这种方法类似于传统的 C 风格寄存器操作，并且由于涉及潜在风险，需要在 Rust 中使用 `unsafe` 块。我们不会涉及这个领域；我还没见过有人使用这种方法。

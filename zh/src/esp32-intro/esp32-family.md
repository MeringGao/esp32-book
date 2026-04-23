# ESP32 系列

如果你是新手，正在寻找该买哪款 ESP32，你可能会被众多的选择和变体所淹没。在本节中，我们将探讨这些变体。

ESP32 系列是由乐鑫（Espressif）推出的一系列低成本、低功耗的片上系统（SoC, System on a Chip）微控制器（Microcontroller）。ESP32 是 ESP8266 的继任者，此后又扩展出了各种新的变体。


## 片上系统（SoC）变体

ESP32 系列包含以下常用的 SoC 变体：

- ESP32
- ESP32 S 系列（ESP32-S2, ESP32-S3）
- ESP32 C 系列（ESP32-C3, ESP32-C6, ESP32-C5）
- ESP32 H 系列（ESP32-H2）
- ESP32-P 系列（ESP32-P4）

如果你正在为特定项目或产品做选择，可以使用上方的产品对比表来找到最适合你需求的型号。你可以在乐鑫的 [产品对比页面](https://products.espressif.com/#/product-comparison) 上找到这些型号的完整列表及其规格。

如需了解最常见 ESP32 型号及其主要功能的简单概述，请查看 Done.land 网站上的 [这个表格](https://done.land/components/microcontroller/families/esp/esp32/)。

## 片上系统（SoC）vs 模块（Module）vs 开发板（Devkit）

ESP32 有三种不同的形式：**片上系统（SoC）**、**模块（Module）** 和 **开发板（Devkits）**。每种形式都有特定的用途，适用于不同的开发或集成阶段。

<a href ="./images/devkit-module-soc.png"><img style="display: block; margin: auto;" src="./images/devkit-module-soc.png"/></a>


### 片上系统（SoC）

SoC 是 ESP32 最核心、最基础的形式。片上系统（SoC）将电子系统或计算机的基本组件集成到单个集成电路（IC, Integrated Circuit）中。就 ESP32 SoC 而言，它包括：

- **CPU**
- **Wi-Fi 和蓝牙**
- **ROM 和 SRAM**
- **其他外设（Peripheral）**

**使用场景**：
SoC 主要用于集成到自定义硬件设计中。对于希望将 ESP32 功能嵌入到其产品中的制造商来说，它是理想的选择。然而，由于单独的 SoC 未通过法规合规性预认证，在设计用于无线通信的自定义硬件时，你需要获得认证（例如 FCC、CE）。

---

### 模块（Module）

模块（Module）在 SoC 的基础上提供了更友好、即插即用的解决方案。常见的 ESP32 模块包括流行的 **WROOM** 和 **WROVER** 系列。例如，**ESP32-WROOM-32** 被广泛使用，在许多开发板上都可以认出它是那个金属外壳的方形元件。

#### **为什么选择模块？**
- **预认证**：模块已通过法规合规性预认证，节省时间和精力。
- **集成组件**：它们包含 PCB 天线、晶振（Crystal Oscillator）和闪存（Flash Memory）芯片。
- **屏蔽设计**：模块带有 EMI 屏蔽罩，可减少电磁干扰（EMI）。

**使用场景**：
模块（Module）设计用于集成到自定义 PCB 中，非常适合需要更少外部组件和更少工作量（与使用原始 SoC 相比）的应用。

---

### 开发板（Devkit）

开发板（Development Board）简化了 ESP32 模块在原型设计和开发中的使用。它们包含额外的组件，使初学者和有经验的开发者都能更轻松地使用 ESP32 模块。

#### **开发板的主要特性**：
- **USB 接口**：便于编程和调试。
- **稳压器（Voltage Regulator）**：为 ESP32 提供稳定的运行电压。
- **引脚引出（Pin Breakouts）**：使 ESP32 的引脚易于访问，以便连接传感器和显示器等外部组件。
- **启动和复位按钮**：提供对模块操作的控制。

**使用场景**：
开发板（Devkit）非常适合快速原型设计和实验。流行的例子包括 **ESP32 DevKit v1** 和 **ESP32-S3-DevkitC**，它们被开发者和爱好者广泛使用。

---
## 为什么选择 ESP32 DevKit v1？

**坦白说：** 当我第一次想尝试 ESP32 时，我并没有意识到有这么多变体可用。我在一个电商网站上搜索，大多数结果显示的是带有不同厂商名称的 ESP32-WROOM-32。我就随便选了一个。后来，我发现还有其他变体。然而，在撰写本文时，我所在地区的大多数其他变体要么不容易买到，要么比这个变体更贵。目前它仍然是一个受欢迎的选择。

因此，在本书中，我们将保持简单，选择受欢迎且价格实惠的 "**ESP32 DevKit V1**"，非常适合开发和学习。


## 如何找到？

如果你在电商网站上搜索，你很可能会发现它列在类似 "ESP32 Development Board (ESP-WROOM-32)" 的名称下，规格中应该标有 WiFi Bluetooth Dual Core (30 PIN)。

你可以用这个来对比规格和板载引脚。

<a href ="../images/esp32-devkitv1.jpg"><img style="display: block; margin: auto;width:300px;" src="../images/esp32-devkitv1.jpg"/></a>

### 规格
ESP32 是一款配备 Wi-Fi 和蓝牙的双核 32 位处理器（Processor），非常适合创建无线物联网（IoT）应用。

以下是 ESP32 的基本规格：
- 处理器（Processor）：Xtensa 32-bit LX6
- 核心数：2
- 时钟频率（Clock Frequency）：240MHz
- 闪存（Flash Memory）：4 MB
- ROM：448 KB（只读程序，对 ESP32 的运行至关重要）
- SRAM：520 KB（用于存储数据和指令）
- ADC：12-bit SAR ADC，18 通道，6 个输入引脚
- UART：3 个
- SPI：2 个
- I2C：3 个
- Wi-Fi：IEEE 802.11 b/g/n/e/i（802.11n 最高 150 Mbps）
- 蓝牙：v4.2 BR/EDR 和低功耗蓝牙（BLE, Bluetooth Low Energy）
- 工作电压：2.3-3.6V
- 深度睡眠：100uA

## 参考
- [Buyers Guide](https://eitherway.io/posts/esp32-buyers-guide/)
- [Chip Series Comparison](https://docs.espressif.com/projects/esp-idf/en/v5.0/esp32s3/hw-reference/chip-series-comparison.html)

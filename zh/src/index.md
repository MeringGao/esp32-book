# 简介

> [Read in English (阅读英文版)](https://esp32.implrust.com/)

在本书中，我们将使用 ESP32 DevKit V1 和 Rust 来构建简单有趣的项目。ESP32 是一款广受欢迎的物联网（IoT）微控制器（Microcontroller），我们采用动手实践（hands-on）的方式，帮助你在实践中学习。你将探索如何使用光敏电阻（LDR）在天色变暗时点亮 LED、使用超声波传感器检测物体靠近、通过 Wi-Fi 控制 LED、在 OLED 显示屏上绘制图像和文字、使用蜂鸣器播放歌曲和警报声、控制舵机（Servo Motor）等等。

我们将使用 Rust 的 `no_std` 环境。虽然可以在 std 环境下为 ESP32 编程，但我认为从 no_std 开始更好，因为这能让你在与其他微控制器（Microcontroller）打交道时应用相同的逻辑。

## 前置要求

如果你还没有阅读过["The Rust on ESP Book"](https://docs.espressif.com/projects/rust/book/)，我强烈建议你先读一读。虽然本书会涵盖开发环境搭建和基础概念的一些方面，但不会过于深入，以避免不必要的重复，因为这些主题在官方书籍中已经有详尽的解释。

我还建议你阅读["The Embedded Rust book"](https://docs.rust-embedded.org/book/intro/index.html)——一本关于在"裸机（Bare Metal）"嵌入式系统上使用 Rust 的入门书籍。

## 认识硬件

我们将使用开发板 "ESP32 DevKit V1"，它内置 Wi-Fi 和蓝牙功能，并集成了射频（RF）模块。
<a href ="./images/esp32-devkitv1.jpg"><img style="display: block; margin: auto;width:300px;" src="./images/esp32-devkitv1.jpg"/></a>


## 数据手册（Datasheet）

如需详细的技术信息、规格和指南，请参阅官方数据手册（Datasheet）：
- [ESP32 Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf)
- [Technical Reference Manual](https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf)
- [ESP32-WROOM-32 Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32_datasheet_en.pdf)

## 面包板（Breadboard）

ESP32 开发板比普通的要稍宽一些。如果你使用标准面包板（Breadboard），可能会像我一样难以安装。为了解决这个问题，我买了两块迷你面包板（Breadboard），将 ESP32 放在它们中间，然后将两边连接起来。

<img style="display: block; margin: auto;width:300px;" src="./images/esp32-devkit-breadboard.png"/>

图片中，下半部分展示了我如何将 ESP32 连接到两块面包板（Breadboard）上。上半部分只是展示了连接之前的面包板（Breadboard）。

## 许可证

"impl Rust for ESP32" 书籍（本项目）采用以下许可证分发：

* 本书中包含的代码示例和独立的 Cargo 项目根据 [MIT License] 和 [Apache License v2.0] 的条款进行许可。
* 本书中的文字内容根据知识共享 [CC-BY-SA v4.0] 许可证的条款进行许可。

[MIT License]: https://opensource.org/licenses/MIT
[Apache License v2.0]: http://www.apache.org/licenses/LICENSE-2.0
[CC-BY-SA v4.0]: https://creativecommons.org/licenses/by-sa/4.0/legalcode


## 支持本项目

你可以通过在 [GitHub](https://github.com/ImplFerris/esp32-book) 上为本项目点星或与他人分享本书来支持它。

### 免责声明：
本书中分享的实验和项目对我本人有效，但结果可能因人而异。对于你在实验过程中可能遇到的任何问题或损坏，我不承担责任。请谨慎操作并采取必要的安全预防措施。

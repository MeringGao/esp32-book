# Ratatui

你可能熟悉 Ratatui，这个流行的 Rust 库用于构建酷炫的终端用户界面（Terminal User Interface，TUI）。许多开发者使用这个 crate 创建令人印象深刻的 TUI 应用程序。但你可能想知道为什么我们在一本嵌入式 Rust 书中讨论 TUI 库。

答案是 [Mousefood](https://github.com/j-g00da/mousefood)。它是一个 embedded-graphics 后端（backend），将 Ratatui 带到微控制器（microcontroller）上。这意味着你可以在任何可以使用 embedded-graphics 的地方运行 Ratatui（例如 TFT 显示屏、电子纸和其他显示屏）。

## 为什么在嵌入式系统中使用 Ratatui？

Ratatui 使构建嵌入式界面比从头编写要容易和快速得多。你可以获得现成的控件（widget），如图表、表格和进度条，而无需复杂的绘制代码。

## 学习 Ratatui 基础

如果你是 Ratatui 的新手，快速浏览一下官方网站 [https://ratatui.rs/](https://ratatui.rs/)。它有清晰的文档和适合初学者的教程，带你创建简单的 TUI 应用程序，如 JSON 编辑器。理解 Ratatui 的核心概念将帮助你将它们适配到嵌入式显示屏上。

## 在嵌入式系统中使用 Ratatui 的项目

以下是一些在嵌入式系统中使用 Ratatui 的有趣项目：

### [Tuitar](https://github.com/orhun/tuitar)

Tuitar 是由 [Orhun Parmaksız](https://github.com/orhun/) 创建的吉他训练工具，它在 ESP32 硬件上独立运行，提供来自音频输入的实时可视化，用于为吉他和其他乐器调音。

Tuitar 可以实时跟踪你在吉他上弹奏的音符，并将它们显示在虚拟指板上。

<img style="display: block; margin: auto;" src="./images/tuitar-embedded-rust-ratatui.jpg" alt="Tuitar"/>

### [适用于 ESP32 CYD 的现代手机操作系统](https://github.com/Julien-cpsn/Phone-OS)

![适用于 ESP32 CYD 的手机操作系统](phone-os-embedded-rust-ratatui-mousefood.png)


### [汽车显示屏：Rust 中的 Suzuki 串行数据线（SDL）查看器](https://github.com/thatdevsherry/suzui-rs)

注意：这个项目没有使用 mousefood crate，也不是真正的嵌入式 Ratatui。代码在 Raspberry Pi 上的标准 Linux 环境中运行。

<img style="display: block; margin: auto;" src="./images/embedded-rust-ratatui-car-display.jpeg" alt="Ratatui in Car Display"/>

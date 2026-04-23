# OLED 模块的工作原理

我们不会深入探讨 OLED 技术的工作原理，而是聚焦于与我们的练习相关的内容。该模块的分辨率为 128x64，总共有 128 × 64 = 8192 个像素（pixel）。每个像素都可以独立开启或关闭。

在数据手册（datasheet）中，128 列被称为段（segment），而 64 行则被称为公共端（common，注意不要因其拼写与 "column" 相似而混淆）。

## 显存

OLED 显示屏的像素在 GDDRAM（Graphics Display DRAM，图形显示动态随机存取存储器）中以页（page）结构排列。GDDRAM 被划分为 8 个页（从 Page 0 到 Page 7），每个页包含 128 列（段，segment）和 8 行（公共端，common）。

<img style="display: block; margin: auto;" title="oled display" src="./images/gddram-oled-display-page-structure.png"/>
（图片来自数据手册）

一个段（segment）是 8 位数据（即一个字节），每一位代表一个像素。写入数据时，你会写入整个段，也就是说整个字节一次性写入。

<img style="display: block; margin: auto;" title="oled display" src="./images/gdram-single-page.png"/>
（图片来自数据手册）

我们可以通过软件重新映射段和公共端，以获得机械布局上的灵活性。更多细节请参阅 SSD1306 数据手册的第 25 页。

## 页与段

我制作了一张图来展示 128x64 像素如何被划分为 8 个页。然后我聚焦于单个页，它包含 128 个段（列）和 8 行。最后，我放大单个段来演示它如何表示 8 个垂直堆叠的像素，每个像素对应一位。

<img style="display: block; margin: auto;" title="oled display" src="./images/oled-pixels-segment-page.png"/>

## 库

如果目前这些概念还不够清晰，不用担心——你可以稍后自行研究。这些细节在你计划为 SSD1306 编写自己的驱动程序或进行更高级的任务时更为相关。目前，我们已经有一个很棒的 Rust crate 可以处理这些方面并简化整个过程。

# 使用 ESP32 驱动 OLED 显示屏

到目前为止，我们都是将输出打印到系统控制台。虽然这对调试来说已经足够，但对于实际应用场景并不理想。如果你需要读取传感器数据（例如温度），或者向用户（或你自己）显示信息，显示屏才是更好的选择。常用的选择有 LCD 和 OLED。

OLED 显示屏比 LCD 更省电、性能更好，因为它不需要背光（backlight）。与 LCD 相比，OLED 提供更高的对比度（contrast）和更出色的图像质量。在本节中，我们将学习如何在 ESP32 上使用 OLED 显示屏模块。

## 认识硬件

OLED，全称 Organic Light-Emitting Diode（有机发光二极管），是一种流行的显示模块。这些显示屏有多种尺寸，并支持不同的颜色。它们通过 I²C 或 SPI 协议进行通信。

在本练习中，我们将使用一块 0.96 英寸的单色（monochrome）OLED 模块，分辨率为 128 x 64。它的工作电压为 3.3V，我们可以使用 I2C 通信协议与之通信。

<img style="display: block; margin: auto;" title="oled display" src="./images/oled.jpg"/>

注意：大多数情况下，OLED 显示屏会附带排针，但并未焊接。焊接（soldering）是一项值得学习的宝贵技能，但它需要细心和准备。在尝试焊接之前，请观看大量教程并做好研究。一开始可能会觉得有挑战性，但熟能生巧。如果你还不熟悉焊接，可以考虑购买预先焊接好的版本，尽管价格可能会稍高一些。

### SSD1306

SSD1306 是一款集成控制芯片（controller chip），许多小型 OLED 显示屏都采用它，包括我们将要使用的模块（0.96 英寸 128x64 模块）。该控制器负责处理 ESP32 与 OLED 面板之间的通信，使显示屏能够显示文本、图形等内容。

**数据手册（DataSheet）：** 你可以在[这里](https://cdn-shop.adafruit.com/datasheets/SSD1306.pdf)找到 SSD1306 的数据手册。

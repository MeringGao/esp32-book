# 在 OLED 显示屏上绘制原始图像

在本练习中，我们将仅使用字节数组来绘制原始图像。我们将以 1BPP（1 Bit Per Pixel，每像素 1 位）格式创建欧姆（Ohm，Ω）符号。

## 1BPP 图像

1BPP（1 bit per pixel，每像素 1 位）格式使用单个位来表示每个像素。它只能表示两种颜色，通常是黑色和白色。如果位值为 0，通常显示为纯黑色；如果位值为 1，通常显示为纯白色。

我们将使用 8x5 像素的网格以 1bpp 格式创建欧姆符号。我在字节数组中高亮显示了 1 的位置，以展示它们如何点亮像素来形成欧姆符号。

<img style="display: block; margin: auto;" title="ohm symbol 1bpp image format" src="../images/embedded-graphics-image-illustration-1bpp.png"/>

我选择宽度为 8 是为了让示例保持简单。这样可以很容易地用单个字节（8 位）来表示 8 像素的宽度。但如果你增加宽度，就无法再容纳在一个字节中了，因此需要分散到字节数组的多个元素中。我将在后续章节中解释这一点。现在，让我们保持简单。

## OLED 显示屏上的欧姆符号（128x64）

让我展示一下当欧姆符号位于 OLED 显示屏（128x64 分辨率）的零位置（x 为 0，y 也为 0）时的样子。

<img style="display: block; margin: auto;" title="ohm symbol in 128x64 pixel" src="../images/resistance-128x64-oled.png"/>

这是一个放大的示意图。当你在实际显示模块上看到该符号时，它会很小。

## 参考资料

- [Embedded Graphics' ImageRaw Documentation](https://docs.rs/embedded-graphics/latest/embedded_graphics/image/struct.ImageRaw.html)
- [Image2Bytes](https://implferris.github.io/image2bytes/)：将图像转换为十六进制字节数组

# 工作原理（How it works?）

液晶显示屏（Liquid Crystal Display，LCD）利用液晶来控制光线。当通电时，液晶改变方向，要么让光线通过，要么阻挡光线，从而形成图像或文字。背光（backlight）照亮屏幕，彩色子像素（红、绿、蓝）组合形成各种颜色。液晶还可以在特定区域变得不透明，阻挡背光，形成暗区以显示字符。

## 16x2 LCD 显示屏与 5x8 像素矩阵（Pixel Matrix）

16x2 LCD 有 2 行和 16 列，总共可以显示 32 个字符。每个字符由 5x8 像素网格组成，其中 5 列和 8 行像素构成字符的形状。这个网格用于在屏幕上显示文字和简单符号。

<img style="display: block; margin: auto;" alt="lcd1602" src="./images/lcd1602-pixel-layout.png"/>

## 在 16x2 LCD 上显示文字和自定义字符（Custom Characters）

我们不需要手动绘制像素；这由 HD44780 IC 自动处理，它会将 ASCII 字符（ASCII character）映射到 5x8 像素网格。

但是，如果你想创建自定义字符（custom character）或符号，则需要自己定义 5x8 像素图案（pixel pattern）。这个图案保存在 LCD 的存储器中，一旦定义完成，你就可以使用该自定义字符。请记住，一次最多只能存储 8 个自定义字符。

## 数据传输模式（Data Transfer Mode）

液晶模块（Liquid Crystal Module，LCM）支持两种数据传输模式：8 位和 4 位。在 8 位模式下，数据作为完整字节通过所有数据引脚发送。在 4 位模式下，仅使用高阶数据位，以半字节（nibble）形式发送数据。虽然 8 位模式更快，但它有一个缺点：需要使用太多导线，这会很快耗尽微控制器（microcontroller）上的 GPIO 引脚。为了减少接线，我们将使用 4 位模式。

## 调整对比度（Contrast）

当你给 LCD 通电时，你应该能看到点阵。如果运行程序后文字不清晰，只需调节 I2C 接口上的电位器（potentiometer）来调整对比度（contrast）。

<img style="display: block; margin: auto;" alt="lcd1602" src="./images/lcd-i2c-pot.png"/>

## 参考：

- [16x2 字符 LCD 点阵模块](http://www.efton.sk/curious/lcd1602.htm)

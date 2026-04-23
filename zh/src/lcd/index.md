# LCD 显示屏（LCD Display）

在本节中，我们将使用日立 HD44780 兼容的 LCD（Liquid Crystal Display，液晶显示屏）。你可能在打印机、数字时钟、微波炉、洗衣机、空调和其他家用电器中见过它们。它们也用于复印机、传真机和路由器等设备中。

你可以显示 ASCII 字符（ASCII character）以及最多 8 个自定义字符（custom character）。

## 型号变体（Variants）

LCD 有多种型号变体，例如 16x2（16 列，2 行）和 20x4（20 列，4 行），还可以根据背光颜色（蓝色、黄色或绿色）区分。我手头的这款显示白色字符，配有蓝色背光。不过，你可以选择任意型号，因为这对代码影响不大。大多数这些型号都有 16 个引脚。

### I2C 型号变体

某些型号带有 I2C（Inter-Integrated Circuit）接口适配器，因此可以使用 I2C 进行通信。I2C 型号的主要优点是减少了引脚连接数量。我们将使用带有 I2C 接口适配器的 LCD 显示屏。

如果你对使用并行接口（Parallel Interface）LCD 感兴趣，可以参考本书 Pico 版本中的 [LCD 章节](https://pico.implrust.com/lcd-display)。但请注意，并行接口需要更多的 GPIO 引脚（但更便宜）。

<img style="display: block; margin: auto;width:400px;" alt="lcd1602 I2C" src="./images/lcd1602-i2c.jpg"/>

因此，在购买 LCD 模块时，请确保它只有四个引脚并包含 I2C 接口适配器，如图所示。

## 硬件需求

我们需要一块 LCD1602 显示屏。推荐使用带 I2C 适配器的 16x2 模块，这样你可以直接跟着操作而无需调整，尽管其他尺寸的显示行为相同。

### 电平转换器（Level Shifter）

<img style="display: block; margin: auto;width:400px;" alt="lcd1602 I2C" src="./images/4 Channel (I2C or SPI) 3.3V-5V Bi-Directional Logic Level Converter.jpg"/>

ESP32 GPIO 引脚的耐压值为 3.6 V，这意味着它们不能安全地用于 5 V 信号。将更高的电压（例如 5 V）施加到这些引脚可能会损坏开发板。许多带 I2C 适配器的 LCD1602 显示屏设计为在 5 V 下工作，当直接连接到 Pico 时会产生电压不匹配的问题。

为了安全地连接 ESP32 和 LCD，我们需要处理这个电压差异。这时就需要使用电平转换器（level shifter）。双向 I2C 逻辑电平转换器（bidirectional I2C logic level shifter）允许 3.3 V 和 5 V 设备安全通信，并保护 ESP32 GPIO 引脚。这些模块价格低廉，通常以"4 通道（I2C）3.3V-5V 双向逻辑电平转换器"的名称销售。

或者，你也可以用 3.3 V 为 LCD 供电。这样可以避免电压问题，但显示屏的背光和对比度会明显变暗。

## 数据手册（Datasheet）

- 你可以从 [Sparkfun](https://www.sparkfun.com/datasheets/LCD/HD44780.pdf) 或 [MIT 网站](https://academy.cba.mit.edu/classes/output_devices/44780.pdf) 获取 HD44780 的数据手册（datasheet）
- [LCD 驱动数据手册](https://www.crystalfontz.com/controllers/datasheet-viewer.php?id=433)
- [LCD 模块 1602A 数据手册](https://www.openhacks.com/uploadsproductos/eone-1602a1.pdf)

## 摇杆移动与对应的 ADC 值

当你沿 X 轴或 Y 轴移动摇杆时，它会产生一个模拟信号（analog signal），电压在 0 到 3.3V 之间变化（如果我们连接到 5V 电源，则为 0 到 5V）。当摇杆处于中心（静止）位置时，输出电压约为 1.65V，即 VCC 的一半（在我们的例子中 VCC 为 3.3V）。

> [!Note]
> 中心位置为 1.65V 的原因是电位器（potentiometer）充当分压器（voltage divider）。当电位器移动时，其电阻发生变化，导致分压器相应地输出不同的电压。请参考<a href="/core-concepts/voltage-divider.html">分压器章节</a>

摇杆共有 5 个引脚，我们稍后会讨论每个引脚代表什么。其中，两个引脚专门用于发送 X 轴和 Y 轴的位置，应连接到微控制器的 ADC 引脚。

如你所知，ESP32 具有 12 位 ADC（Analog-to-Digital Converter，模数转换器），可将模拟信号（电压差）转换为数字值。由于它是 12 位 ADC，模拟值将被表示为 0 到 4095 之间的数字值。如果你不熟悉 ADC，请参考我们之前介绍的 [ADC 章节](../core-concepts/adc.md)。

<img style="display: block; margin: auto;width:580px;" alt="joystick-movement" src="./images/joystick-movement-and-corresponding-esp32-adc-values.jpg"/>

**注意：**

图片中的 ADC 值只是近似值，给你一个大致的概念，不会完全精确。例如，我在中心位置时 X 和 Y 约为 1850。当我将摇杆帽向引脚图一侧移动时，X 变为 0，当我将其移到相反一侧时，变为 4095。Y 轴也是如此。所以，你可能需要校准你的摇杆。

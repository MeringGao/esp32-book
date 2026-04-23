# 热敏电阻（Thermistor）

在本节中，我们将使用热敏电阻（thermistor）配合 ESP32 进行实验。热敏电阻是一种根据温度变化而改变阻值的可变电阻（variable resistor）。阻值变化的大小取决于其材料成分。这个词由 "thermal"（热）和 "resistor"（电阻）组合而来。

热敏电阻在许多地方被用于测量温度或保护电路。它们帮助测量车辆中的机油和冷却液温度，也被用于医疗设备中。在智能家居中，热敏电阻存在于智能恒温器中，用于控制加热和制冷，使温度保持在合适的范围。我们将使用热敏电阻来测量室温并显示结果。

热敏电阻分为两类：

- **NTC（Negative Temperature Coefficient，负温度系数）**：
  - 温度升高时，阻值降低。
  - 主要用于温度传感和浪涌电流限制。
  - 我们将在练习中使用 NTC 热敏电阻来测量温度。
  
  <img style="display: block; margin: auto;" alt="pico2" src="./images/ntc-resistor.png"/>

- **PTC（Positive Temperature Coefficient，正温度系数）**：
  - 温度升高时，阻值增加。
  - 主要用作可复位保险丝，防止过流和过温，常见于空调、医疗设备、电池充电器和焊接设备中。

## 硬件需求

- NTC 103 热敏电阻：10K 欧姆，5mm 环氧树脂涂层圆片
- 10kΩ 电阻：与热敏电阻配合使用构成分压器（voltage divider）

## 工作原理

我们将使用 NTC 热敏电阻和一个已知阻值的电阻（10kΩ）来创建一个分压器。随着温度升高，热敏电阻的阻值会降低。然后我们将根据这个阻值计算温度。在下一章中，我们将展示热敏电阻在分压器中的仿真以及它产生的电压。

然而，当热敏电阻连接到微控制器时，你无法直接测量其阻值。相反，你会得到来自分压器的输出电压，该电压被送入微控制器的 ADC 引脚。这个电压被读取为 ADC 值（更多细节请参阅 [ADC 章节](../core-concepts/adc/index.md)）。然后我们将使用公式从 ADC 值计算出阻值。

## 参考资料

- [Thermistor Basics](https://www.teamwavelength.com/thermistor-basics/)
- [Thermistors](https://www.electronics-tutorials.ws/io/thermistors.html)

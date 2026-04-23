## LDR（光敏电阻，Light Dependent Resistor）

在本节中，我们将使用 LDR（光敏电阻，也称为 photocell 或 photoresistor）与 ESP32。LDR 的阻值会根据照射在它上面的光线量而变化。光线越亮，阻值越低；光线越暗，阻值越高。这使它非常适合用于光线感应、自动照明或监测环境光照水平等应用。

<img style="display: block; margin: auto;" alt="pico2" src="./images/ldr.png"/>


## 所需元件：
- LDR（光敏电阻，Light Dependent Resistor）
- 电阻（通常为 10kΩ）；需要用它来构成分压器（Voltage Divider）
- 跳线（像往常一样）

## 前置知识

要处理这个，你应该先熟悉什么是分压器以及它的工作原理。你还需要了解什么是 ADC 以及它的功能。
- [分压器章节](../core-concepts/voltage-divider.md)
- [ADC](../core-concepts/adc/index.md)

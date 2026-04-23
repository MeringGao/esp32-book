## NTC 与分压器

我在 Falstad 网站上创建了一个电路，你可以下载 [voltage-divider-thermistor.circuitjs.txt](./voltage-divider-thermistor.circuitjs.txt) 文件来导入并进行实验。这个设置与我们在 [分压器](../core-concepts/voltage-divider.md) 章节中介绍的内容类似。如果你还没有阅读过那一节，我强烈建议你在继续之前先完成那里的理论学习。

该电路包含一个 10kΩ 的热敏电阻（R2），在 25°C 时阻值为 10kΩ。输入电压 \( V_{in} \) 设置为 3.3V。

**注意：** 当你使用微控制器创建这个分压器时，ADC 外设（peripheral）会将 Vout 转换为数字值。

### 25°C 时的热敏电阻

热敏电阻在 25°C 时阻值为 10kΩ，输出电压（\( V_{out} \)）为 1.65V。

<img style="display: block; margin: auto;" alt="pico2" src="./images/thermistor0.png"/>

## 38°C 时的热敏电阻

由于热敏电阻的负温度系数（negative temperature coefficient），其阻值降低，从而改变了分压器的输出。

<img style="display: block; margin: auto;" alt="pico2" src="./images/thermistor1.png"/>

## 10°C 时的热敏电阻

热敏电阻的阻值增加，导致输出电压（\( V_{out} \)）更高。

<img style="display: block; margin: auto;" alt="pico2" src="./images/thermistor2.png"/>

这些仅用于说明目的；你的热敏电阻的实际测量值可能不完全相同。

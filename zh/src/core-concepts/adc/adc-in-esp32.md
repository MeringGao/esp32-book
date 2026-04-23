## ESP32 中的模数转换器（Analog to Digital Converter, ADC）

ESP32 配备了两颗 12 位 SAR ADC（逐次逼近寄存器模数转换器），支持最多 18 个测量通道。然而，我们使用的开发板只有 15 个 ADC 引脚，因此仅支持 15 个通道。

第一颗（ADC1）提供 8 个通道，映射到 GPIO32 至 GPIO39。第二颗（ADC2）提供 10 个通道，映射到 GPIO0、GPIO2、GPIO4、GPIO12 至 GPIO15，以及 GPIO25 至 GPIO27。

你可以在[技术参考手册](https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf#page=639&zoom=100,76,1020)中进一步了解 ESP32 的 ADC。

> [!Important]
> **Wi-Fi 与 ADC2**：如果你在 ESP32 上使用 Wi-Fi，则不能使用 ADC2 引脚，因为它们被 Wi-Fi 模块占用。你只能使用 ADC1 引脚。

### ESP32 DevKit V1 的 ADC 引脚
[![ADC ESP32 DevKit V1 引脚图](../images/ESP32-DevKit-V1-Pinout-Diagram-adc-pins.png)](../images/ESP32-DevKit-V1-Pinout-Diagram-adc-pins.png)

### ESP32 中的 ADC 参考电压与衰减

ADC 需要一个参考电压（Vref）来与输入电压进行比较，以便计算输入电压的数字值（范围从 0 到 4095，因为它是 12 位 ADC）。这个参考电压称为 Vref（参考电压），帮助 ADC 将输入电压映射到这个范围内。

ESP32 使用的 Vref 约为 1.1V。这意味着它只能映射 0V 到 1.1V 之间的输入电压。但如果输入电压高于 1.1V 会怎样？这就是衰减（attenuation）发挥作用的地方。

衰减（attenuation），简单来说，就是降低某个值。在我们的情况下，它帮助将较高的输入电压映射到 Vref 的范围内。这样，ADC 就可以将大于 1.1V 的电压映射到 0 到 4095 的范围内。

在 `esp-hal` 中，这表示为 `Attenuation` 枚举；我们需要在代码中配置它才能使用 ADC。

ESP32 支持四个衰减等级：

| **衰减等级（Attenuation Level）** | **esp-hal 中的枚举**         | **可测量的输入电压范围** |
|-----------------------|------------------------------|-----------------------------------|
| 0 dB                  | Attenuation::Attenuation0dB   | 100 mV ~ 950 mV                  |
| 2.5 dB                | Attenuation::Attenuation2p5dB | 100 mV ~ 1250 mV                 |
| 6 dB                  | Attenuation::Attenuation6dB   | 150 mV ~ 1750 mV                 |
| 11 dB                 | Attenuation::Attenuation11dB  | 150 mV ~ 2450 mV                 |


ESP32 的 ADC 已知存在非线性问题。然而，由于我们的大部分练习不需要高精度，我们不会深入这些细节。

## 参考
- [Espressif API 参考（旧版）](https://docs.espressif.com/projects/esp-idf/en/v4.4/esp32s3/api-reference/peripherals/adc.html)

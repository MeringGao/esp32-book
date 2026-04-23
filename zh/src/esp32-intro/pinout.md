# ESP32 引脚图（Pinout）

[![ESP32 DevKit V1 引脚图（Pinout Diagram）](./images/ESP32-DevKit-V1-Pinout-Diagram.png)](./images/ESP32-DevKit-V1-Pinout-Diagram.png)

许可证：[CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.en)

上面的引脚图（Pinout Diagram）改编自 CircuitState 网站创建的原始图表。他们还提供了每个引脚的详细说明。你可以在 [这里](https://www.circuitstate.com/pinouts/doit-esp32-devkit-v1-wifi-development-board-pinout-diagram-and-reference/) 查看。

免责声明：本书与 CircuitState 没有关联关系。我包含此图表是因为我在研究过程中发现它很有帮助。

## 要点

- **仅输入 GPIO：** GPIO 引脚 34、35、36 和 39 仅支持输入（Input-Only），不能用作输出引脚。在引脚图（Pinout Diagram）中，这些引脚标有 "GPIX" 前缀，并带有 "X" 标记以表示不允许输出。只要可能，请优先使用图中以紫色高亮显示的引脚。

- **烧录（Flashing）和调试：** GPIO 1 (Tx) 和 GPIO 3 (Rx) 用于烧录（Flashing）和调试。

- **ADC 引脚：** 标有 `ADC1_[Number]` 的引脚对应 ADC1，而标有 `ADC2_[Number]` 的引脚对应 ADC2。

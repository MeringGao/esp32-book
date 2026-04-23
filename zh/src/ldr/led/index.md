## 在光线较暗时点亮 LED（或灯）

在本练习中，我们将根据环境光照水平控制 LED。目标是在低光照条件下自动打开 LED。

你可以在关闭房间灯光的封闭房间里尝试这个实验。当你关闭房间灯时，LED 应该会亮起（前提是房间足够暗），当你重新打开房间灯时，LED 会再次关闭。或者，你可以调整灵敏度阈值，或者用手或某个物体遮住光线传感器（LDR）来模拟不同的光照水平。

注意：你可能需要根据房间的光照条件和你使用的具体 LDR 来调整 ADC 阈值。

## 硬件需求

- **LED** – 任何标准 LED（选择你喜欢的颜色）。
- **LDR（光敏电阻，Light Dependent Resistor）** – 用于检测光照强度。
- **电阻**
  - **330Ω** – 用于 LED 以限制电流并防止损坏。（你可能需要根据你的 LED 进行选择）
  - **10kΩ** – 用于 LDR，在电路中构成分压器。
- **跳线** – 用于在面包板或微控制器上连接元件。


## 连接 LED、LDR 与 ESP32 的电路

**连接 LDR 与 ESP32**：

1. **LDR 的一侧**连接到地线
2. **LDR 的另一侧**连接到 **GPIO 4 (ADC2)**（即 Devkit 上的 D4）
3. 一个**电阻**与 LDR 串联，在 LDR 和 **3v3** 之间构成分压器。

<img style="display: block; margin: auto;" alt="circuit connection of esp32 with photoresistor" src="../images/connection-esp32-with-photoresistor-circuit.png"/>

**连接 LED 与 ESP32**：

这是我们之前做过的常规设置。你需要将 LED 的阴极（短脚）连接到地线。然后将阳极（长脚）与 330 欧姆电阻串联后连接到 GPIO 33（即 Devkit 上的 D33）。





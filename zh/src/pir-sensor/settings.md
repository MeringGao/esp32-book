
## 调整 PIR 传感器的设置

<img style="display: block; margin: auto;width:60vh;" alt="HC-SR501" src="./images/hc-sr501-details.jpg"/>
<p>图片来源：<a href="https://learn.adafruit.com/pir-passive-infrared-proximity-motion-sensor/overview">Adafruit</a></p>

PIR 传感器模块带有两个内置电位器（Potentiometers），允许你微调其行为：延时时间（Delay Time）和灵敏度（Sensitivity）。这些小小的调节旋钮让你可以控制传感器对运动的响应方式，使其更适应不同的环境和应用。

### 延时时间（Delay Time / Output Duration）

此设置决定传感器在检测到运动后输出保持高电平（HIGH）的时间。较长的延时适用于自动照明等应用，你希望灯光在检测到运动后保持亮一段时间。

- **顺时针旋转**以增加持续时间（最长约 200 秒）。
- **逆时针旋转**以减少持续时间（最短约 5 秒）。

### 灵敏度（Sensitivity / Detection Range）

此设置控制传感器能检测到运动的距离。较高的灵敏度允许传感器从更远的距离捕捉到运动，而较低的灵敏度会缩小检测范围，以防止在小区域内产生误触发。

- **顺时针旋转**以扩大范围（最远约 7 米）。
- **逆时针旋转**以缩短范围（最近约 3 米）。

**注意：**

说实话，我们即将编写的代码非常简单。对我来说最具挑战性的部分是弄清楚旋转方向。我知道是顺时针（和逆时针），但我不确定传感器应该朝上还是朝下。关键是将传感器圆顶朝上拿着，然后按照顺时针和逆时针的说明进行调节。

## 跳线设置（Jumper Setting）

除了电位器旋钮，你还可以通过跳线设置进行调整。它们允许你在两种模式之间切换：重复触发（Retriggering）和非重复触发（Non-Retriggering）。

<img style="display: block; margin: auto;width:60vh;" alt="HC-SR501" src="./images/hc-sr501-retrigger-setting-jumper.jpg"/>

### 重复触发模式（Retriggering Mode / H）

在此模式下，只要检测到运动，输出就保持高电平。如果在计时器结束前有更多运动发生，它会重置。非常适合像有人进入房间时灯亮着，只要有运动就保持亮着的场景。

### 非重复触发模式（Non-Retriggering Mode / L）

在此模式下，一旦检测到运动，输出就保持高电平，但在延时时间结束之前不会再次触发。非常适合像计算通过门口的人数这样的场景，每次检测都需要是一个独立的事件。

### 如何设置跳线

要切换模式，只需将跳线（那个小的黑色/黄色方块——是的，你可以把它取下来放在两个引脚之间）移到 H 引脚以实现重复触发，或移到 L 引脚以实现非重复触发。

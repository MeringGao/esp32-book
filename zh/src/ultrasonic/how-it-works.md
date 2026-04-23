# 超声波传感器的工作原理

超声波传感器通过发射频率过高（40kHz）而人耳无法听到的声波来工作。这些声波在空气中传播，遇到物体时会反射回来。传感器通过测量声波返回所需的时间来计算距离。

<img style="display: block; margin: auto;width:500px" alt="ultrasonic" src="./images/ultrasonic.jpg"/>

- **发射器（Transmitter）：** 发射超声波。
- **接收器（Receiver）：** 检测从物体反射回来的声波。

**距离计算公式：**
```
距离 = (时间 x 声速) / 2
```

在标准大气压和 20°C 温度下，声速约为 0.0343 厘米/微秒（或 343 米/秒）。

## 计算示例：

假设超声波传感器检测到声波从物体反射回来用了 2000 微秒。

第一步：计算声波传播的总距离：
```
总距离 = 时间 x 声速
总距离 = 2000 µs x 0.0343 cm/µs = 68.6 cm
```

第二步：由于声波传播到物体并返回，所以到物体的距离是总距离的一半：
```
到物体的距离 = 68.6 cm / 2 = 34.3 cm
```

因此，物体距离传感器 34.3 厘米。

## HC-SR04 引脚图（Pinout）

该模块有四个引脚：VCC、Trig、Echo 和 GND。

<table style="width:300px;height:200px;">
<tr>
    <th>引脚</th>
    <th>功能</th>
</tr>
<tr>
    <td style="vertical-align: middle;text-align: center;" class="slanted-text st-red">VCC</td>
    <td>电源（Power Supply）</td>
</tr>
<tr>
    <td style="vertical-align: middle;text-align: center;" class="slanted-text st-yellow">Trig</td>
    <td>触发信号（Trigger Signal）</td>
</tr>
<tr>
    <td style="vertical-align: middle;text-align: center;" class="slanted-text st-teal">Echo</td>
    <td>回响信号（Echo Signal）</td>
</tr>
<tr>
    <td style="vertical-align: middle;text-align: center;" class="slanted-text st-blue">GND</td>
    <td>地线（Ground）</td>
</tr>
</table>

## 使用 HC-SR04 模块测量距离

HC-SR04 模块有一个发射器和一个接收器，分别负责发送超声波和检测反射波。我们将使用 Trig 引脚来发送声波，并从 Echo 引脚读取数据以测量距离。

<img style="display: block; margin: auto;" alt="ultrasonic" src="./images/ultrasonic-trigger-echo-wave.png"/>

如上图所示，我们将 Trig 和 Echo 引脚连接到微控制器的 GPIO 引脚（同时连接 VCC 和 GND，但为了简化图示省略了）。我们通过将 Trig 引脚置为高电平（HIGH）10 微秒，然后将其恢复为低电平（LOW）来发送超声波。这会触发模块以 40 kHz 的频率连续发送 8 个超声波脉冲。建议每次触发之间至少间隔 50 毫秒。

当传感器的声波遇到物体时，会反射回模块。如上图所示，Echo 引脚会改变输入到微控制器的信号，信号保持高电平的时间长度（脉冲宽度，Pulse Width）对应着距离。在微控制器中，我们测量 Echo 引脚保持高电平的时间；然后，我们可以利用这段时间来计算到物体的距离。

**脉冲宽度与距离的关系：**

Echo 引脚产生的脉冲宽度（保持高电平的时间）范围约为 150 微秒到 25,000 微秒（25 毫秒）；这仅在声波击中物体时成立。如果没有物体，它将产生约 38 毫秒的脉冲宽度。

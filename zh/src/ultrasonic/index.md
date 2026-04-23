# HC-SR04 超声波传感器（Ultrasonic Sensor）

在本指南中，我们将学习如何在 ESP32 上使用 HC-SR04 超声波传感器。超声波传感器通过发射超声波并计算声波从物体反射回来所需的时间来测量距离。

这类传感器常见于汽车倒车辅助系统；当你倒车入库时，传感器会测量与障碍物之间的距离，并在你靠近时发出警报。

我们将构建一个简单的项目：当超声波传感器检测到物体距离小于 30 厘米时，使用 PWM 逐渐增加 LED 的亮度——你可以根据需要调整这个阈值。

<img style="display: block; margin: auto;width:500px" alt="ultrasonic" src="./images/hc-sr04-ultrasonic.jpg"/>

## 前置知识

在开始之前，请先熟悉以下主题：

- [PWM](../core-concepts/pwm/index.md)

## 🛠 硬件需求

要完成本项目，你需要：

- HC-SR04 超声波传感器（或 HC-SR04+）
- 面包板（Breadboard）
- 跳线（Jumper wires）
- 外接 LED（你也可以使用板载 LED，只需将引脚换成 GPIO 2）

## HC-SR04 规格

HC-SR04 超声波传感器的测量范围为 2 厘米到 400 厘米，精度最高可达 3 毫米。它需要 5V 电源供电。

- [数据手册（Datasheet）](https://cdn.sparkfun.com/datasheets/Sensors/Proximity/HCSR04.pdf)

## ESP32 耐压值（GPIO Tolerance）

在下一章中，我们将深入了解传感器的工作原理，但基本思路很简单：它发送一个信号并等待接收返回的信号。当使用 5V 供电时，传感器的 Echo 引脚会向连接的设备输出 5V 信号。但这里有个问题，ESP32 的 GPIO 引脚最高只能承受 3.6V。任何更高的电压都有可能损坏微控制器。

**如何解决这个问题？**

以下是几种可选方案：
- 最简单的方案是购买 HC-SR04+，这是升级版本，支持 3.3V 和 5V 两种供电电压。然后使用 3.3V 为模块供电。
- 另一种方案是在 Echo 引脚和 ESP32 之间使用一个分压器（Voltage Divider）（只需几个电阻），将电压降到 3.3V。
- 最后一种方案是使用 3.3V 为 HC-SR04 供电。它可能会工作，但可靠性不高。除非万不得已，否则不建议使用此方案，并且你需要承担任何潜在的副作用。

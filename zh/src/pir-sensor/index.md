# PIR 传感器（PIR Sensor）

你有没有走进一个房间，灯就自动亮了？那可能就是 PIR 传感器在起作用。

在之前的章节中，我们使用[超声波传感器](../ultrasonic/index.md)来检测人或物体是否在传感器附近。它通过发送超声波并测量到物体的距离来工作。然而，PIR 传感器的工作原理不同；它不是测量距离，而是检测运动。

PIR 传感器被称为"被动"红外（Passive Infrared），因为它不发射任何红外辐射；相反，它只检测环境中红外辐射的变化。它能感知人、动物和其他温暖物体发出的热量。当检测区域内发生移动时，传感器会捕捉到变化并发送信号。这使它适用于自动照明、防盗报警和其他运动检测系统。

## 认识硬件

我们将使用 **HC-SR501** PIR 传感器模块。它内置了一个热释电传感器（Pyroelectric Sensor），帮助检测运动，以及一个圆顶形的菲涅尔透镜（Fresnel Lens），帮助扩大检测范围。它可以使用 5V 到 12V 的电源供电。

PIR 传感器模块有三个引脚：一个用于电源，一个用于输出（中间引脚），一个用于地线。它很容易使用，因为它提供了一个简单的输出。默认情况下，当没有检测到运动时，信号保持低电平（LOW）。但当有人在检测范围内移动时，输出会跳变为高电平（HIGH），表示检测到运动。

<img style="display: block; margin: auto;width:400px" alt="HC-SR501" src="./images/pir-sensor-HC-SR501.jpg"/>

## 资源

PIR 传感器模块基于 BISS0001 控制器构建。以下是一些与 BISS0001 控制器相关的数据手册。

- [https://cdn-learn.adafruit.com/assets/assets/000/010/133/original/BISS0001.pdf](https://cdn-learn.adafruit.com/assets/assets/000/010/133/original/BISS0001.pdf)
- [http://www.sc-tech.cn/en/BISS0001.pdf](http://www.sc-tech.cn/en/BISS0001.pdf)

如果你想了解 PIR 传感器的内部工作原理，可以查看这篇指南：
[How PIRs Work](https://learn.adafruit.com/pir-passive-infrared-proximity-motion-sensor/how-pirs-work)。我们不会在这本书中介绍这部分内容，因为它超出了本书的范围。我们的重点仅仅是检测运动并将信号发送给 ESP32。

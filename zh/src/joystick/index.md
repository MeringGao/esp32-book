# 摇杆（Joystick）

在本节中，我们将探索如何使用摇杆模块（Joystick Module）。它与 PS2（PlayStation 2）控制器上的摇杆类似。它们常用于游戏，以及控制无人机、遥控车、机器人和其他需要调整位置或方向的设备。

## 认识硬件——摇杆模块

<img style="display: block; margin: auto;width:250px;" alt="joystick" src="./images/joystick.jpg"/>

你可以垂直和水平移动摇杆帽（knob），它会将位置信息（X 轴和 Y 轴）发送给 MCU（如 ESP32）。此外，摇杆帽可以像按钮一样按下。摇杆通常工作在 5V，但也可以连接到 3.3V。

## 工作原理？

摇杆模块有两个 10K 电位器（potentiometer）：一个用于 X 轴，另一个用于 Y 轴。它还包括一个可见的按钮（push button）。

当你将摇杆从左到右或从右到左移动（X 轴）时，你可以观察到其中一个电位器随之移动。同样，当你上下移动（Y 轴）时，你可以观察到另一个电位器随之移动。

<img style="display: block; margin: auto;width:550px;" alt="joystick" src="./images/joystick-potentiometers-push-button.jpg"/>

当你向下按压摇杆帽时，你还可以观察到按钮被按下。

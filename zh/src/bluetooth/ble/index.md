# BLE

要使用蓝牙低功耗（Bluetooth Low Energy, BLE），我们需要了解几个关键概念。我会尽量简洁，只涵盖足够让你入门的内容，而不会用太多细节让你感到不知所措。所以，系好安全带，让我们开始吧。

## BLE 协议栈

下图展示了蓝牙低功耗（BLE）协议栈。BLE 协议栈是 BLE 设备之间通信的基础。我们不会详细介绍控制器（Controller，下层），因为它对我们的目的来说不是必需的。然而，理解主机（Host）部分的关键概念，如 GAP 和 GATT，是很重要的。

<img style="display: block; margin: auto;" alt="Bluetooth LE protocol stack" src="../images/ble-stack.jpg"/>

### GAP => 设备如何连接和通信

GAP（Generic Access Profile，通用访问配置文件）定义了 BLE 设备如何广播（advertise）、连接和建立通信。它涵盖设备角色（如中心设备 central、外设 peripheral）、连接参数和安全模式。GAP 负责设备如何发现彼此并启动通信。

### GATT => 设备如何交换和结构化数据

GATT（Generic Attribute Profile，通用属性配置文件）定义了 BLE 设备如何交换数据。它以服务（services）和特征（characteristics）的层次结构组织数据，允许客户端（如智能手机应用）读取、写入和订阅来自 BLE 外设（如传感器）的更新。

## Rust crate

有两个主要的 Rust crate 可用于蓝牙低功耗（BLE）：

1. [Bleps](https://github.com/bjoernQ/bleps/)：一个轻量级 BLE 外设协议栈，主要用于测试、演示和个人项目。它不打算满足认证标准。官方仓库建议，如果你计划使用异步代码，请使用 `Trouble` crate。

2. [Trouble（或 TrouBLE）](https://embassy.dev/trouble/)：一个基于 Rust 的 BLE 主机（Host）实现，长期目标是获得资格认证。在 BLE 规范中，主机在主机控制器接口（HCI）中作为上层运行，而控制器形成下层。

## 参考资料

如果你想更深入地理解，可以参考以下资源：
- [Download The Bluetooth Low Energy Primer](https://www.bluetooth.com/bluetooth-resources/the-bluetooth-low-energy-primer/)
- [Bluetooth Low Energy Fundamentals](https://academy.nordicsemi.com/courses/bluetooth-low-energy-fundamentals/)

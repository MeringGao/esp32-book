# 属性协议（ATT）和通用属性配置文件（GATT）

在上一章中，我们了解到 GAP 层帮助蓝牙 LE 设备通过广播发现彼此。连接后，它们需要一种发送和接收数据的方式。这就是 ATT 和 GATT 层的作用；它们定义了数据如何在设备之间结构化和传输。

## 客户端-服务器模型

GATT 中有两个角色：服务器（Server）和客户端（Client）。服务器将数据保存为属性（attributes），客户端访问这些数据。通常，外设设备（如传感器）充当服务器，中心设备（如智能手机）充当客户端。

> [!Note]
> **中心设备-外设 vs 服务器-客户端**：GATT 中的客户端和服务器角色与通用访问配置文件（GAP）中的外设和中心设备角色是独立的。这意味着中心设备可以是客户端或服务器，外设设备也是如此。

例如，在智能手机和健身追踪器的场景中，健身追踪器（外设）通常充当 GATT 服务器，存储心率或步数等传感器数据，而智能手机（中心设备）充当 GATT 客户端，读取这些数据以在应用中显示。然而，如果智能手机需要向追踪器发送配置设置（例如调整显示亮度或设置闹钟），它会临时成为服务器，而健身追踪器则充当客户端来接收这些设置。

## 属性协议（ATT）——基础

ATT 定义了数据如何作为属性存储；属性是基础和构建块。每个属性都有一个唯一的句柄（handle）、类型（16 位标识符或 128 位 UUID）、权限（如可读、可写）和数据（实际值）。客户端可以读取、写入或订阅数据。

## 通用属性配置文件（GATT）——组织数据

GATT 在 ATT 的基础上增加了结构和含义。它定义了数据如何分组和访问。

GATT 将属性组织为：

- **特征（Characteristic）**：设备可以共享的单个数据片段。其他设备可以读取、写入或接收其更新。例如，心率测量特征保存当前心率，并可在其变化时发送更新。

- **服务（Service）**：一组相关的特征组合在一起。例如，心率服务包括心率测量和传感器在身体位置的定位等特征。

- **配置文件（Profiles）**：一组相关的服务（例如，心率服务、设备信息服务）。

下图展示了心率传感器的配置文件、服务和特征：

<img style="display: block; margin: auto;" alt="GAT" src="../images/ble-gatt.png"/>

### UUID

让我们重新审视属性中的 UUID 部分。每个服务和特征都应该有一个唯一的 ID 值。UUID 可以是标准蓝牙 SIG 定义的 UUID（16 位）或自定义 UUID（128 位）。

你可以从这里获取预定义的 UUID 列表：[https://www.bluetooth.com/specifications/assigned-numbers/](https://www.bluetooth.com/specifications/assigned-numbers/)

心率服务的预定义 UUID：
<img style="display: block; margin: auto;" alt="GAT" src="../images/heart-rate-service.png"/>

心率监测特征的预定义 UUID：
<img style="display: block; margin: auto;" alt="GAT" src="../images/heart-rate-monitor-characteristics.png"/>

**自定义 UUID**：

在我们的示例中，我们将使用自定义 UUID 而不是预定义的。但是，如果你要实现像心率传感器这样的通用服务，最好使用官方文档中列出的预定义 UUID。

要生成自定义 UUID，你可以访问 [UUID Generator](https://www.uuidgenerator.net/) 并为你的服务和特征创建唯一的 UUID。

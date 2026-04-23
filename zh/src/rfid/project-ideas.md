# 项目创意

## 1. 使用 RFID 和 OLED 的门禁系统

构建一个门禁系统，当扫描 RFID 标签时，在 OLED 显示屏上显示 "Access Granted"（允许访问）或 "Access Denied"（拒绝访问）。可选地，添加一个蜂鸣器（buzzer）根据访问状态提供音频反馈。

### 组件

- MFRC522（RFID 读卡器模块）& RFID 标签/卡片
- 0.96 英寸 OLED 显示屏（I2C 接口）
- [可选] 蜂鸣器（用于音频反馈）
- 电源（例如 5V 适配器或电池）

### 相关学习资源

- [OLED 显示屏](../oled/index.md)
- [蜂鸣器](../buzzer/index.md)

## 2. 使用 RFID 的自动车库门

我发现了一个有趣的视频，演示了使用 RFID 和 Arduino 实现的自动车库门系统。你可以使用 ESP32 和 Rust 复刻这个项目。[点击这里观看视频](https://www.youtube.com/watch?v=ICnYGbvkrpo)。

### 组件

- MFRC522（RFID 读卡器模块）& RFID 标签/卡片
- 舵机（servo motor）
- [可选] 蜂鸣器（可选，用于音频反馈）
- 电源（例如 5V 适配器或电池）

### 相关学习资源

- [舵机](../servo/index.md)

## 3. 使用 ESP32 和 RFID 的简单智能门锁系统

可以使用纸板箱制作一个门模型，当出示正确的钥匙时，舵机（servo motor）会打开门。如果使用了错误的钥匙，门保持关闭，可选地，蜂鸣器会发出反馈声音。你可以使用 ESP32 和 Rust 复刻这个项目。[点击这里观看视频](https://www.youtube.com/watch?v=3xb2PLFjJxk)。

### 组件

- MFRC522（RFID 读卡器模块）& RFID 标签/卡片
- 舵机（servo motor）
- [可选] 蜂鸣器（可选，用于音频反馈）
- 电源（例如 5V 适配器或电池）

### 相关学习资源

- [舵机](../servo/index.md)

### 作品展示

如果你基于这个创意创建了一个项目，欢迎发送拉取请求（pull request）附上你的项目链接，我们会在这里展示！

- [ESP32 RFID Access Control](https://github.com/implferris/esp32-rfid-access)：使用 Rust 和 ESP32 实现的智能门锁模拟，使用 RFID、可选舵机和 OLED 显示屏来模拟和控制门禁。

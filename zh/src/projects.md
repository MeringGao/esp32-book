# 为 ESP32 编写的 Rust 项目精选列表

以下是我在网上找到的与 ESP32 和 Rust 相关的有趣项目精选列表。如果你有一些有趣的项目想要展示，请发送 PR :)

注意：它将包含与所有 ESP32 系列相关的项目。因此可能不是确切的 ESP32 SoC。

## ESP32
- [ESP32 Rex](https://github.com/implferris/esp32-rex)：使用 Embassy 框架为 ESP32 开发的恐龙游戏，配 OLED 显示屏。
- [ESP32 RFID 门禁控制](https://github.com/implferris/esp32-rfid-access)：使用 Rust 和 ESP32 的智能门锁模拟，使用 RFID、可选舵机和 OLED 显示屏来模拟和控制门禁。
- [ESP32 Wi-Fi 坦克](https://github.com/jamesmcm/esp32_wifi_tank)：使用 ESP32 控制板构建的 Wi-Fi 控制坦克/漫游车
- [太阳能逆变器](https://github.com/Orange-Murker/solar_inverter)：带 MPPT 的并网太阳能逆变器
- [纸质列车](https://github.com/vhdirk/papertrain)：在由 esp32 驱动的电子墨水屏上显示 NMBS 列车延误
- [贪吃蛇游戏](https://jamesmcm.github.io/blog/beginner-rust-esp32-lcdsnake/)：编写一个贪吃蛇游戏在 ESP32 开发板上运行，连接 OLED 显示屏并用摇杆控制

## ESP32C3
- [ha-vfd-dashboard](https://github.com/Orange-Murker/ha-vfd-dashboard/) 使用真空荧光显示屏制作的家庭助手仪表盘
- [ESP32-C3 Embassy](https://github.com/claudiomattera/esp32c3-embassy)：通过 I²C 从 BME280 传感器采样环境数据（温度、湿度、气压），并通过 SPI 在 WaveShare 1.54 英寸 B 型 v2 电子墨水屏上显示最新样本。

## ESP32-S3
- [风速计](https://github.com/taunusflieger/anemometer)：基于 ESP32-S3 Espressif 的 3D 打印风速计

## 通用
- [esp32-spooky-maze-game](https://github.com/georgik/esp32-spooky-maze-game)：用于 ESP32 的迷宫游戏 Rust 裸机实现

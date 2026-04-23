# 电路 - 连接 ESP32 与热敏电阻

1. 热敏电阻的一侧连接到 `GND`。
2. 热敏电阻的另一侧连接到 `GPIO13`，这是 ESP32 的 ADC2 引脚。
3. 一个 10kΩ 的电阻与热敏电阻串联，在热敏电阻和 `3.3v` 之间形成一个分压器。因此，将电阻的一侧连接到 `3.3V`，另一侧连接到 `GPIO13`。

<img style="display: block; margin: auto;" alt="esp32 与热敏电阻的电路连接" src="./images/esp32-thermistor.png"/>

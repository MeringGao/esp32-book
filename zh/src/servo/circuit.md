
## 连接概述
1. **地线（GND）**：将舵机的 GND 引脚（通常是**棕色**线，但可能因型号而异）连接到 ESP32 上的任意地线引脚。
2. **电源（VCC）**：将舵机的 VCC 引脚（通常是**红色**线）连接到 ESP32 的 5V（Vin）电源引脚。
3. **信号（PWM）**：将舵机的控制（信号）引脚连接到 ESP32 的 **GPIO 33**，并配置为 PWM。这通常是**橙色**线（可能因型号而异）。

<table style="margin-bottom:20px">
  <thead>
    <tr>
      <th>ESP32 引脚</th>
      <th style="width: 250px; margin: 0 auto;">导线</th>
      <th>舵机</th>
      <th>说明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>VIN</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>电源（红色线）</td>
      <td>为舵机提供 5V 电源。</td>
    </tr>
    <tr>
      <td>GND</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire brown" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>地线（棕色线）</td>
      <td>连接到地线。</td>
    </tr>
    <tr>
      <td>GPIO 33</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire yellow" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>信号（橙色线）</td>
      <td>接收 PWM 信号以控制舵机位置。</td>
    </tr>
  </tbody>
</table>

<img style="display: block; margin: auto;" alt="connecting servo motor with esp32" src="./images/connecting-servo-motor-sg90-with-esp32.png"/>

# 电路

我们将使用 I2C 在 ESP32 和 OLED 显示屏之间进行通信。对于 I2C，需要将两个 GPIO 引脚（pin）配置为 SDA（Serial Data，串行数据）和 SCL（Serial Clock，串行时钟）。在 ESP32 上，我们可以使用任意 GPIO 引脚作为 I2C。这里我们将 GPIO 18 配置为 SCL，GPIO 23 配置为 SDA。OLED 的 VCC 引脚连接到 ESP32 的 3.3V 引脚，GND 引脚连接到 ESP32 的地线（ground）。

<table style="margin-bottom:20px">
  <thead>
    <tr>
      <th>ESP32 引脚</th>
      <th style="width: 250px; margin: 0 auto;">连线</th>
      <th>OLED 引脚</th>
      <th>说明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>3.3V</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>VCC</td>
      <td>为 OLED 显示屏供电。</td>
    </tr>
    <tr>
      <td>GND</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire black" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>Ground</td>
      <td>连接到 OLED 的地线引脚。</td>
    </tr>
    <tr>
      <td>GPIO 18</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire blue" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>SCL</td>
      <td>连接 I2C 通信的时钟信号（SCL）。</td>
    </tr>
    <tr>
      <td>GPIO 23</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire yellow" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>SDA </td>
      <td>连接 I2C 通信的数据信号（SDA）。</td>
    </tr>
  </tbody>
</table>

<img style="display: block; margin: auto;" title="esp32 oled circuit" src="./images/connecting esp32 with oled ssd1306 circuit.png"/>

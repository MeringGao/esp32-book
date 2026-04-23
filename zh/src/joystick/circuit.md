# 将摇杆连接到 ESP32

让我们将摇杆连接到 ESP32。我们需要将 VRX 和 VRY 引脚连接到 ESP32 的 ADC 引脚。摇杆将使用 3.3V 供电而不是 5V，因为 ESP32 的 GPIO 引脚只能承受 3.3V。将其连接到 5V 可能会损坏 ESP32 的引脚。幸运的是，摇杆也可以在 3.3V 下工作。

<table>
  <thead>
    <tr>
      <th>ESP32 引脚</th>
      <th style="width: 250px; margin: 0 auto;">导线</th>
      <th>摇杆引脚</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GND</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire black" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GND</td>
    </tr>
    <tr>
      <td>3.3V</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>VCC</td>
    </tr>
    <tr>
      <td>GPIO 13</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire green" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>VRX</td>
    </tr>
    <tr>
      <td>GPIO 14</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire blue" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>VRY</td>
    </tr>
    <tr>
      <td>GPIO 32</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire yellow" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>SW</td>
    </tr>
  </tbody>
</table>

<img style="display: block; margin: auto;margin-top:30px;" alt="connecting joystick with esp32" src="./images/connecting joystick with esp32.png"/>

## HC-SR501 引脚图（Pinout）

输出引脚很容易辨认，因为它就在中间。

如果你不确定哪个引脚是地线（GND），哪个是电源（VCC），你可以取下白色圆顶来检查。你也可以通过寻找保护二极管旁边的引脚来识别——那就是 VCC 引脚。

## 将 PIR 传感器与 ESP32 连接

该传感器工作在 5V，但 ESP32 的 GPIO 引脚只能承受 3.3V。PIR 传感器模块在检测到运动时输出 3.3V，因此可以直接连接到 GPIO，不会有任何问题。否则，你需要一个分压器（Voltage Divider）将电压降到 3.3V。

<table style="margin-bottom:20px">
  <thead>
    <tr>
      <th>ESP32 引脚</th>
      <th style="width: 250px; margin: 0 auto;">导线</th>
      <th>PIR 传感器引脚</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GPIO 33</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire yellow" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>OUT（中间引脚）</td>
    </tr>
    <tr>
      <td>5V</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>VCC</td>
    </tr>
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
  </tbody>
</table>
<br/>
<img style="display: block; margin: auto;" alt="HC-SR501" src="./images/esp32-pir-sensor-connection-circuit.png"/>

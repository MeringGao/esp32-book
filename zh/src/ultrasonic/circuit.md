## 将 HC-SR04 超声波传感器与 ESP32 连接

### 为什么需要分压器（Voltage Divider）？

如果你使用的是支持 3.3V 供电的 HC-SR04+，可以跳过本节。

在连接之前，你应该先熟悉分压器的概念；你可以参考[这一章](../core-concepts/voltage-divider.md)。正如我们之前提到的，我们需要使用 5V 电源为模块供电，这可以通过 ESP32 的 Vin 引脚提供。模块通过 Echo 引脚发回信号。然而，ESP32 的 GPIO 引脚最高只能承受约 3.6V，因此我们需要在 Echo 引脚和 ESP32 的 GPIO 引脚之间使用一个分压器来降低电压。

要实现这一点，你需要两个阻值不同的电阻，确保输出电压约为 3.3V。例如，你可以使用 1kΩ 电阻作为 R1，2kΩ 电阻作为 R2，这样可以将电压降到约 3.3V。

<img style="display: block; margin: auto;" alt="ultrasonic" src="./images/voltage-divider-hc-sr04-3_3_v.png"/>

如果你想尝试不同的电阻值，可以使用 [Falstad 网站](https://www.falstad.com/circuit/)配合这个[分压电路文本文件](./voltage-divider-hc-sr04.txt)。它允许你修改 R1 和 R2 的值，查看每种组合会输出什么电压。这样，你就可以用手头现有的电阻来制作分压器。

## HC-SR04 电路

<table>
  <thead>
    <tr>
      <th>ESP32 引脚</th>
      <th style="height: 4px; width: 250px; margin: 0 auto;">导线</th>
      <th>HC-SR04 引脚</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Vin (5V)</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="height: 4px; width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>VCC</td>
    </tr>
    <tr>
      <td>GPIO 5</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire green" style="height: 4px; width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>Trig</td>
    </tr>
    <tr>
      <td>GPIO 18</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire yellow" style="height: 4px; width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>Echo（通过分压器）</td>
    </tr>
    <tr>
      <td>GND</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire black" style="height: 4px; width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GND</td>
    </tr>
  </tbody>
</table>

- **VCC**：将 HC-SR04 的 VCC 引脚连接到 ESP32 的 Vin 引脚。如果你使用的是 HC-SR04+，请使用 ESP32 的 3.3V 引脚。
- **Trig**：连接到 ESP32 的 GPIO 5。
- **Echo**：将 HC-SR04 的 Echo 引脚通过分压器连接到 GPIO 18（1kΩ 电阻接在 Echo 和 GPIO 18 之间，2kΩ 电阻接在 GPIO 18 和 GND 之间；也就是说 GPIO 18 基本上位于中间）。我相信电路图会让你更容易理解。
- **GND**：将 HC-SR04 的 GND 引脚连接到 ESP32 的 GND 引脚。

## LED 电路

你需要将 LED 的阳极（长脚）通过电阻（例如：330 欧姆电阻）连接到 GPIO 33，如[外接 LED 设置](../led/external-led.md)所示；以避免损坏 LED。并将 LED 的阴极（短脚）连接到地线。

<table>
  <thead>
    <tr>
      <th>ESP32 引脚</th>
      <th style="width: 250px; margin: 0 auto;">导线</th>
      <th>元件</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GPIO 33</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire orange" style="width: 200px; margin: 0 auto;">
          <div class="female-left"></div>
          <div class="female-right"></div>
        </div>
      </td>
      <td>电阻</td>
    </tr>
    <tr>
      <td>电阻</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire orange" style="width: 200px; margin: 0 auto;">
          <div class="female-left"></div>
          <div class="female-right"></div>
        </div>
      </td>
      <td>LED 阳极（长脚）</td>
    </tr>
    <tr>
      <td>GND</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire black" style="width: 200px; margin: 0 auto;">
          <div class="female-left"></div>
          <div class="female-right"></div>
        </div>
      </td>
      <td>LED 阴极（短脚）</td>
    </tr>
  </tbody>
</table>

## 电路图

我提供了带面包板和不带面包板两种电路图。说实话，带面包板的版本画起来有点混乱。如果你觉得还是不清楚，请在 GitHub 仓库中创建一个 Issue 并描述不清楚的部分。我会尽力改进。

**不带面包板的电路图**：

<img style="display: block; margin: auto;" alt="connecting ESP32 with HC-SR04 Ultrasonic Sensor circuit" src="./images/ESP32-HC-SR04-circuit-without-breadboard.png"/>

**带面包板的电路图**：
<img style="display: block; margin: auto;" alt="connecting ESP32 with HC-SR04 Ultrasonic Sensor circuit" src="./images/ESP32-HC-SR04-circuit.png"/>

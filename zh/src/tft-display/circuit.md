# 将 ESP32 与 TFT 显示屏连接

在本节中，我们将查看 TFT 显示模块的引脚图（pinout），并了解如何将其连接到 ESP32。如果你仔细观察，它与电子墨水屏显示模块非常相似。这是因为两个显示屏都使用 SPI（Serial Peripheral Interface）进行通信，并且共享共同的控制线，如 CS、DC、RESET、MOSI 和 SCK。这意味着一旦你了解如何接线和驱动一个基于 SPI 的显示屏，切换到另一个就会容易得多。

## TFT 显示屏引脚图

<img style="display: block; margin: auto;" src="./images/tft-display-pinout.png" alt="tft display pinout"/>

- **VCC**：电源（3.3V 或 5V）。这是整个显示模块的主电源。它为显示控制器（如 ILI9341）和其他部分供电。
- **GND**：地线（Ground）
- **CS**：片选（Chip Select）。这告诉显示屏何时应该监听 SPI 命令。在发送数据时保持低电平（低电平有效 active low）。
- **RESET**：重置显示屏。在启动期间很有用，以确保显示屏以已知状态启动。
- **DC**：数据/命令控制引脚。设置为高电平以发送数据，低电平以发送命令。用于在写入命令和像素数据之间切换。
- **SDI (MOSI)**：主出从入（Master Out Slave In）。这是从微控制器到显示屏的 SPI 数据线。用于发送像素数据和命令。
- **SCK**：串行时钟（Serial Clock）。来自微控制器的 SPI 时钟信号。它同步正在发送的数据。
- **LED**：这个引脚专门用于显示屏的背光（backlight，LED 面板）。它控制屏幕显示的亮度。如果你希望背光始终开启，只需将其连接到 3.3V。如果你想控制亮度（例如调暗或关闭），可以将其连接到微控制器的 PWM 功能 GPIO 引脚。
- **SDO (MISO)**：主入从出（Master In Slave Out）。从显示屏到微控制器的 SPI 读取线。不总是使用，但某些显示屏支持读取显示内存或状态。


### 触摸控制器引脚图

如果你的 TFT 显示屏包含电阻式触摸屏，它可能使用像 **XPT2046** 这样的触摸控制器芯片。这些触摸引脚与显示引脚分开，也通过 SPI 连接。只有在你想使用触摸功能时才需要连接这些引脚。

- **T_CLK**：触摸屏 SPI 总线时钟引脚
- **T_CS**：触摸屏片选控制引脚
- **T_DIN**：触摸屏 SPI 写入数据引脚（MOSI）
- **T_DO**：触摸屏 SPI 读取数据引脚（MISO）
- **T_IRQ**：触摸屏中断检测引脚

如果你不需要触摸输入，或者你的模块没有触摸面板，你可以将这些引脚保持不连接。


## 将 TFT 显示屏与 ESP32 连接

<img style="display: block; margin: auto;" src="./images/esp32-tft-display.png" alt="tft display with esp32"/>
<br/>

<table style="margin-bottom:20px">
  <thead>
    <tr>
      <th>TFT 显示屏引脚</th>
      <th style="width: 250px; margin: 0 auto;">导线</th>
      <th>ESP32 引脚</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>VCC</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>Vin</td>
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
    <tr>
      <td>CS</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire white" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO15</td>
    </tr>
    <tr>
      <td>RESET</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire brown" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO4</td>
    </tr>
    <tr>
      <td>DC</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire purple" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO2</td>
    </tr>
    <tr>
      <td>SDI (MOSI)</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire green" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO23</td>
    </tr>
    <tr>
      <td>SCK (CLK)</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire blue" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO18</td>
    </tr>
    <tr>
      <td>LED</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire orange" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>3.3V（或用于亮度控制的 PWM 引脚）</td>
    </tr>
  </tbody>
</table>

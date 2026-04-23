# 电路（Circuit）

## 电子纸显示屏引脚图（Pinout）

<img style="display: block; margin: auto;" src="./images/e-paper-1.54 inch-display.png" alt="e-ink pinout"/>

- **VCC**：电源（3.3V 或 5V），为显示屏供电。
- **GND**：地线（Ground）
- **DIN (MOSI)**：SPI 数据线，用于从微控制器向显示屏发送信息。
- **CLK (SCK)**：同步 SPI 数据传输。
- **CS (Chip Select，低电平有效)**：启用或禁用 SPI 通信。当为低电平（LOW）时，显示屏处于活动状态。
- **DC (Data/Command Select)**：确定发送的数据是像素数据（高电平 HIGH）还是命令（低电平 LOW）。
- **RST (Reset，低电平有效)**：当拉低再拉高时重置显示屏。对初始化至关重要。
- **BUSY**：指示显示屏当前是否正在处理（高电平 HIGH）或准备接收新命令（低电平 LOW）。

## 将电子纸显示屏与 ESP32 连接

<img style="display: block; margin: auto;" src="./images/esp32-circuit-e-ink.png" alt="e-ink pinout"/>
<br/>
<table style="margin-bottom:20px">
  <thead>
    <tr>
      <th>电子纸引脚</th>
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
      <td>3.3V</td>
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
      <td>DIN (MOSI)</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire blue" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO23</td>
    </tr>
    <tr>
      <td>CLK (SCK)</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire yellow" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO18</td>
    </tr>
    <tr>
      <td>CS</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire orange" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO33</td>
    </tr>
    <tr>
      <td>DC</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire green" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO17 (TX2 in Devkit)</td>
    </tr>
    <tr>
      <td>RST</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire white" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO16 (Rx2 in Devkit)</td>
    </tr>
    <tr>
      <td>BUSY</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire purple" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO22</td>
    </tr>
  </tbody>
</table>

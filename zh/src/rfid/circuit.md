## 连接 RC522 与 ESP32

## RC522 引脚图

RC522 RFID 模块共有 8 个引脚。

<a href="./images/rc522-pinout.jpg"><img style="display: block; margin: auto;" alt="pinout diagram of RC522" src="./images/rc522-pinout.jpg"/></a>

<table style="border-collapse: collapse; width: 100%; border: 1px solid black;">
  <tr style="border: 1px solid black;">
    <th style="background-color: #009B77; border: 1px solid black;">引脚</th>
    <th style="border: 1px solid black;">SPI 功能</th>
    <th style="border: 1px solid black;">I²C 功能</th>
    <th style="border: 1px solid black;">UART 功能</th>
    <th style="border: 1px solid black;">描述</th>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #ff0000; color: white; border: 1px solid black;">3.3V</td>
    <td style="border: 1px solid black;">电源</td>
    <td style="border: 1px solid black;">电源</td>
    <td style="border: 1px solid black;">电源</td>
    <td style="border: 1px solid black;">电源供电（3.3V）。</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #000; color: white; border: 1px solid black;">GND</td>
    <td style="border: 1px solid black;">地线</td>
    <td style="border: 1px solid black;">地线</td>
    <td style="border: 1px solid black;">地线</td>
    <td style="border: 1px solid black;">接地连接。</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #7B3F00; color: white; border: 1px solid black;">RST</td>
    <td style="border: 1px solid black;">复位</td>
    <td style="border: 1px solid black;">复位</td>
    <td style="border: 1px solid black;">复位</td>
    <td style="border: 1px solid black;">复位模块。</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #EFEFEF; border: 1px solid black;color:black;">IRQ</td>
    <td style="border: 1px solid black;">中断（可选）</td>
    <td style="border: 1px solid black;">中断（可选）</td>
    <td style="border: 1px solid black;">中断（可选）</td>
    <td style="border: 1px solid black;">中断请求（IRQ）在检测到 RFID 标签时通知微控制器。如果不使用 IRQ，微控制器需要不断轮询（poll）模块。</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #FFC000; color: black; border: 1px solid black;">MISO</td>
    <td style="border: 1px solid black;">Master-In-Slave-Out</td>
    <td style="border: 1px solid black;">SCL</td>
    <td style="border: 1px solid black;">TX</td>
    <td style="border: 1px solid black;">在 SPI 模式下，作为 Master-In-Slave-Out（MISO）。在 I²C 模式下，作为时钟线（SCL）。在 UART 模式下，作为发送引脚（TX）。</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #008000; color: white; border: 1px solid black;">MOSI</td>
    <td style="border: 1px solid black;">Master-Out-Slave-In</td>
    <td style="border: 1px solid black;">-</td>
    <td style="border: 1px solid black;">-</td>
    <td style="border: 1px solid black;">在 SPI 模式下，作为 Master-Out-Slave-In（MOSI）。</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #0F52BA; color: white; border: 1px solid black;">SCK</td>
    <td style="border: 1px solid black;">Serial Clock</td>
    <td style="border: 1px solid black;">-</td>
    <td style="border: 1px solid black;">-</td>
    <td style="border: 1px solid black;">在 SPI 模式下，作为同步数据传输的时钟线。</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #FF5F1F; color: white; border: 1px solid black;">SDA</td>
    <td style="border: 1px solid black;">CS（或 SS）</td>
    <td style="border: 1px solid black;">SDA</td>
    <td style="border: 1px solid black;">RX</td>
    <td style="border: 1px solid black;">在 SPI 模式下，作为片选（Chip select, CS，也称为 Slave Select）。在 I²C 模式下，作为数据线（SDA）。在 UART 模式下，作为接收引脚（RX）。</td>
  </tr>
</table>

## 将 RFID 读卡器连接到 ESP32

要建立 ESP32 与 RFID 读卡器之间的通信，我们将使用 SPI（Serial Peripheral Interface，串行外设接口）协议。SPI 接口的数据传输速度最高可达 10 Mbit/s。我们暂时不会使用以下引脚：RST、IRQ。

<table>
  <thead>
    <tr>
      <th>ESP32 引脚</th>
      <th style="width: 250px; margin: 0 auto;">导线</th>
      <th>RFID 读卡器引脚</th>
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
      <td>GPIO 5</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire green" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>标为 SDA，在 SPI 模式下作为 CS 引脚。</td>
    </tr>
    <tr>
      <td>GPIO 18</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire blue" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>SCK</td>
    </tr>
    <tr>
      <td>GPIO 19</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire yellow" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>MISO</td>
    </tr>
    <tr>
      <td>GPIO 23</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire orange" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>MOSI</td>
    </tr>
  </tbody>
</table>

<img style="display: block; margin: auto;" alt="connecting RC522 with ESP32" src="./images/connecting-esp32-with-rfid-rc522.png"/>

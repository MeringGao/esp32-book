## 电路

### microSD 卡 SPI 模式引脚映射

我们将只关注 microSD 卡，因为这是我们正在使用的。microSD 有 8 个引脚，但在 SPI 模式下我们只需要 6 个。你可能已经注意到，我们拥有的 SD 卡读卡器模块也只有 6 个引脚，并标有 SPI 功能标识。下表显示了 microSD 卡引脚及其对应的 SPI 功能。

<div style="display: flex; align-items: center;gap:18px;">
  <img style="width: 180px;" alt="microSD Card Pin Diagram" src="./images/micro-sd-card-pin.png"/>
  <table>
    <thead>
      <tr>
        <th>microSD 卡引脚</th>
        <th>SPI 功能</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>1</td>
        <td>-</td>
      </tr>
      <tr>
        <td>2</td>
        <td>片选（Chip Select，CS）；也称为卡选择（Card Select）</td>
      </tr>
      <tr>
        <td>3</td>
        <td>数据输入（Data Input，DI）- 对应 MOSI。用于从微控制器接收数据。</td>
      </tr>
      <tr>
        <td>4</td>
        <td>VDD - 电源（3.3V）</td>
      </tr>
      <tr>
        <td>5</td>
        <td>串行时钟（Serial Clock，SCK）</td>
      </tr>
      <tr>
        <td>6</td>
        <td>地线（Ground，GND）</td>
      </tr>
      <tr>
        <td>7</td>
        <td>数据输出（Data Output，DO）- 对应 MISO。用于从 microSD 卡向微控制器发送数据。</td>
      </tr>
      <tr>
        <td>8</td>
        <td>-</td>
      </tr>
    </tbody>
  </table>
</div>

### 将 ESP32 连接到 SD 卡读卡器

microSD 卡的工作电压为 3.3V，因此使用 5V 供电可能会损坏卡片。然而，读卡器模块带有板载稳压器（onboard voltage regulator）和电平转换器（logic shifter），使其可以安全地连接到 ESP32 的 5V 电源。

<table>
  <thead>
    <tr>
      <th>ESP32 引脚</th>
      <th style="width: 250px; margin: 0 auto;">连线</th>
      <th>SD 卡引脚</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GPIO 5</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire green" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>CS</td>
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
      <td>GPIO 23</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire orange" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>MOSI</td>
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
      <td>Vin</td>
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
<img style="display: block; margin: auto;" alt="SD Card reader pico connection" src="./images/connecting-micro-sdcard-reader-module-with-esp32-devkit-v1.png"/>

# 将 LCD 显示屏（LCD1602）连接到 ESP32

我们将把一块带有 I2C 适配器的 LCD1602 字符显示屏连接到 ESP32。从接线角度来看，设置看起来很简单，因为只需要四个连接：电源、地线、SDA 和 SCL。然而，尽管接线数量很少，但在进行任何连接之前，我们必须正确处理一个重要的电压细节。

## 电压兼容性问题

ESP32 GPIO 引脚的耐压值（GPIO tolerance）为 3.6 V。如果 GPIO 引脚上的电压超过 3.6 V，则超出了安全工作范围，可能会损坏芯片。

大多数带 I2C 背包（I2C backpack）的 LCD1602 模块设计为在 5 V 下运行。I2C 背包通常有上拉电阻（pull-up resistors）连接到其供电电压。当在 5 V 下供电时，这意味着 SDA 和 SCL 在空闲时处于 5 V。

如果直接将这种模块的 SDA 和 SCL 连接到 ESP32，ESP32 GPIO 引脚将暴露在 5 V 下。这是我们必须解决的核心问题。

## 网上常见的捷径建议

许多在线教程建议用 5 V 为 LCD1602 及其 I2C 背包供电，并直接将 SDA 和 SCL 连接到 ESP32。我自己也测试过这种接法，它确实可以工作。然而，这在电气上不安全，长期使用可能会损坏 ESP32 GPIO 引脚。

因此，尽管它能工作，但这种接线方法不应被视为安全或推荐的方案。

> [!Note]
> 这个话题在 ESP32 社区中经常被讨论。一些用户报告称 5 V 输入在业余项目中似乎可以工作，而另一些人则指出这种行为并未被 Espressif 规定，不应依赖，特别是对于商业或长期设计。其中一个相关讨论可以在这里找到：
> [https://www.reddit.com/r/esp32/comments/1j5p0lb/can_the_espwroom32_read_5v_io_inputs/](https://www.reddit.com/r/esp32/comments/1j5p0lb/can_the_espwroom32_read_5v_io_inputs/)

## 简单但相对安全的方法：全部使用 3.3 V 供电

对于演示、实验和学习项目，最简单且最安全的方法是用 ESP32 的 3.3 V 电源为 LCD1602 I2C 模块供电，而不是 5 V。

当 LCD 背包在 3.3 V 下供电时，其 I2C 上拉电阻（pull-up resistors）将 SDA 和 SCL 拉到 3.3 V 而不是 5 V。这立即消除了电压兼容性问题，SDA 和 SCL 可以直接连接到 ESP32。

代价是 LCD 对比度（contrast）和背光亮度（backlight brightness）降低。在 3.3 V 下供电时，显示屏在正常室内照明条件下通常仍然可读，这对于实验和基本验证已经足够。

这种方法避免了额外的元件，也避免了对 ESP32 GPIO 引脚造成压力。

<table>
  <thead>
    <tr>
      <th style="width: 250px;">LCD 引脚</th>
      <th style="width: 250px; text-align: center;">导线</th>
      <th>ESP32 引脚</th>
      <th>说明</th>
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
      <td>地线（Ground）</td>
    </tr>
    <tr>
      <td>VCC</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>3.3</td>
      <td>3.3 V 电源</td>
    </tr>
    <tr>
      <td>SCL</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire blue" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO 18</td>
      <td>连接 I2C 通信的时钟信号（SCL）。</td>
    </tr>
    <tr>
      <td>SDA</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire green" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO 23</td>
      <td>连接 I2C 通信的数据信号（SDA）。</td>
    </tr>
  </tbody>
</table>
<br/>
<img style="display: block; margin: auto;" alt="lcd1602" src="./images/connecting-esp32-with-lcd-display-lcd1602.png"/>


## 最佳方案：使用电平转换器（Level Shifter）

如果你需要全亮度的背光，或者你的 LCD 模块在 3.3 V 下无法可靠工作，那么你必须用 5 V 为其供电。但这意味着你需要保护 ESP32 免受 5 V 信号的影响。

解决方案是使用双向逻辑电平转换器（bidirectional logic level converter，也称为电平转换器 level shifter）。这个小模块可以在两个方向上将信号在 3.3 V 和 5 V 之间转换。

ESP32 连接到电平转换器的 3.3 V 侧，LCD 连接到 5 V 侧。电平转换器确保 ESP32 始终只接收到 3.3 V 信号，而 LCD 获得它需要的 5 V 信号。

这是在电气上正确且安全的方法，但它增加了几根额外的导线，并需要一个额外的模块。

<br/>
<img style="display: block; margin: auto;" alt="lcd1602" src="./images/connecting-esp32-with-lcd-display-lcd1602-using-level-converter.png"/>

电路图乍一看可能有点令人困惑，但一旦分解开来，思路其实很简单。

<h3>电源连接（先连接这些）</h3>

首先，将 ESP32 的 3.3 V 输出连接到电平转换器上标有 LV（Low Voltage，低电压）的引脚。这定义了 ESP32 侧的逻辑电平（logic level）。接下来，将 ESP32 的 5 V 电源（即 ESP32 DevKit V1 开发板上的 VIN）连接到电平转换器上标有 HV（High Voltage，高电压）的引脚。LCD 的 VCC 引脚也由这个相同的 5 V 电源供电。

地线（Ground）必须是共用的，所以将 ESP32 GND、电平转换器 GND 和 LCD GND 连接在一起。

<table>
  <thead>
    <tr>
      <th style="width: 180px;">从</th>
      <th style="width: 120px;">引脚</th>
      <th style="width: 160px; text-align: center;">导线</th>
      <th style="width: 180px;">到</th>
      <th style="width: 120px;">引脚</th>
      <th>用途</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ESP32</td>
      <td>GND</td>
      <td style="text-align: center; padding: 0;">
        <div class="wire black" style="width: 140px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>电平转换器</td>
      <td>GND</td>
      <td>共用地线</td>
    </tr>
    <tr>
      <td>电平转换器</td>
      <td>GND</td>
      <td style="text-align: center; padding: 0;">
        <div class="wire black" style="width: 140px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>LCD 显示屏</td>
      <td>GND</td>
      <td>共用地线</td>
    </tr>
    <tr>
      <td>ESP32</td>
      <td>3.3V</td>
      <td style="text-align: center; padding: 0;">
        <div class="wire orange" style="width: 140px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>电平转换器</td>
      <td>LV</td>
      <td>低电压逻辑参考</td>
    </tr>
    <tr>
      <td>ESP32</td>
      <td>VIN / 5V</td>
      <td style="text-align: center; padding: 0;">
        <div class="wire red" style="width: 140px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>电平转换器</td>
      <td>HV</td>
      <td>高电压逻辑参考</td>
    </tr>
    <tr>
      <td>ESP32</td>
      <td>VIN / 5V</td>
      <td style="text-align: center; padding: 0;">
        <div class="wire red" style="width: 140px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>LCD 显示屏</td>
      <td>VCC</td>
      <td>LCD 的 5V 电源</td>
    </tr>
  </tbody>
</table>

<h3>数据连接（I2C 通信）</h3>

现在连接 I2C 信号线。ESP32 GPIO 引脚必须始终连接到电平转换器上标有 LVx 的引脚，LCD I2C 引脚必须连接到对应标有 HVx 的引脚。

在这个例子中，ESP32 SDA 引脚连接到 LV1，LCD SDA 引脚连接到 HV1。ESP32 SCL 引脚连接到 LV2，LCD SCL 引脚连接到 HV2。用于 SDA 和 SCL 的具体 GPIO 编号取决于你的 ESP32 开发板和软件配置。

<table>
  <thead>
    <tr>
      <th style="width: 180px;">从</th>
      <th style="width: 120px;">引脚</th>
      <th style="width: 160px; text-align: center;">导线</th>
      <th style="width: 180px;">到</th>
      <th style="width: 120px;">引脚</th>
      <th>用途</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ESP32</td>
      <td>GPIO 23 (SDA)</td>
      <td style="text-align: center; padding: 0;">
        <div class="wire green" style="width: 140px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>电平转换器</td>
      <td>LV1</td>
      <td>SDA（数据）- ESP32 侧</td>
    </tr>
    <tr>
      <td>电平转换器</td>
      <td>HV1</td>
      <td style="text-align: center; padding: 0;">
        <div class="wire green" style="width: 140px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>LCD 显示屏</td>
      <td>SDA</td>
      <td>SDA（数据）- LCD 侧</td>
    </tr>
    <tr>
      <td>ESP32</td>
      <td>GPIO 18 (SCL)</td>
      <td style="text-align: center; padding: 0;">
        <div class="wire blue" style="width: 140px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>电平转换器</td>
      <td>LV2</td>
      <td>SCL（时钟）- ESP32 侧</td>
    </tr>
    <tr>
      <td>电平转换器</td>
      <td>HV2</td>
      <td style="text-align: center; padding: 0;">
        <div class="wire blue" style="width: 140px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>LCD 显示屏</td>
      <td>SCL</td>
      <td>SCL（时钟）- LCD 侧</td>
    </tr>
  </tbody>
</table>


### 工作原理

简单来说，当 ESP32 通过 I2C 线通信时，LVx 侧以 3.3 V 工作，电平转换器在对应的 HVx 侧为 LCD 提供 5 V 信号。当 LCD 以 5 V 通信时，电平转换器将信号转换回来，使 ESP32 在 LVx 侧始终只接收到 3.3 V。这样，两个设备都能在其所需电压下工作，而不会对 ESP32 GPIO 引脚造成压力。

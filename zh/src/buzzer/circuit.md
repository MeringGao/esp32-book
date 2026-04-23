
## 将蜂鸣器与 ESP32 连接

蜂鸣器有两个引脚：正极（信号）和地线（GND）；蜂鸣器的正极通常标有 **+** 符号，且引脚较长，而负极（地线）引脚较短，与 LED 类似。然而，某些无源蜂鸣器可能允许任意一个引脚连接到地线或信号，具体取决于型号。

顺便说一下，我在实验中使用的是有源蜂鸣器。如果你打算播放不同的声音，建议使用无源蜂鸣器，因为它能提供更好的音调。

<table style="margin-bottom:20px">
  <thead>
    <tr>
      <th>ESP32 引脚</th>
      <th style="width: 250px; margin: 0 auto;">导线</th>
      <th>蜂鸣器引脚</th>
      <th>说明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GPIO 33</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>正极引脚</td>
      <td>接收 PWM 信号以产生声音。</td>
    </tr>
    <tr>
      <td>GND</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire black" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>地线引脚</td>
      <td>连接到地线。</td>
    </tr>
  </tbody>
</table>


<img style="display: block; margin: auto;" alt="connecting buzzer with esp32" src="./images/esp32-with-active-buzzer.png"/>

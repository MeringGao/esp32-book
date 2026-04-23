# 引脚布局

摇杆共有 5 个引脚：电源、地线、X 轴输出、Y 轴输出和开关输出引脚。

<img style="display: block; margin: auto;width:400px;margin-bottom: 10px;" alt="joystick" src="./images/joystick-pin-layout.jpg"/>

<table border="1" style="border-collapse: collapse; width: 100%;">
  <thead>
    <tr>
      <th style="width:14%">摇杆引脚</th>
      <th>详情</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><span class="slanted-text black">GND</span></td>
      <td>地线引脚。应连接到电路的地线。</td>
    </tr>
    <tr>
      <td><span class="slanted-text red">VCC</span></td>
      <td>电源引脚（通常为 5V 或 3.3V）。</td>
    </tr>
    <tr>
      <td><span class="slanted-text green">VRX</span></td>
      <td>X 轴模拟输出引脚，根据摇杆的水平位置改变电压，当摇杆左右移动时，电压范围为 0V 到 VCC。</td>
    </tr>
    <tr>
      <td><span class="slanted-text blue">VRY</span></td>
      <td>Y 轴模拟输出引脚，根据摇杆的垂直位置改变电压，当摇杆上下移动时，电压范围为 0V 到 VCC。</td>
    </tr>
    <tr>
      <td><span class="slanted-text purple">SW</span></td>
      <td>开关引脚。当按下摇杆帽时，该引脚通常被拉低（到 GND）。</td>
    </tr>
  </tbody>
</table>


# 脉冲宽度调制（Pulse Width Modulation, PWM）

<style>

  .slider-container {
    margin: 20px 0;
  }

  label {
    margin-right: 10px;
  }

  .led-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2px;
    position: relative;
  }

  .led-body {
    width: 30px;
    height: 40px;
    background: radial-gradient(circle at center, #ff5555, #cc0000);
    border-radius: 50% 50% 0 0;
    border: 2px solid #990000;
    position: relative;
    box-shadow: 0 0 10px rgba(255, 85, 85, 0.8);
  }

  .led-body::after {
    content: '';
    position: absolute;
    top: 5px;
    left: 7px;
    width: 16px;
    height: 16px;
    background: rgba(255, 255, 255, 0.4);
    border-radius: 50%;
  }

  .led-pin {
    width: 2px;
    height: 40px;
    background-color: #333;
    position: relative;
  }

  .anode {
    height: 50px; /* Longer pin for the anode */
    margin-right: 15px;
    background-color: #666;
  }

  .cathode {
    height: 40px; /* Shorter pin for the cathode */
    margin-left: 15px;
    background-color: #333;
    position: absolute;
    margin-top: 45px;
  }

  canvas {
    border: 1px solid #ccc;
    display: block;
    margin: 10px auto;
  }
  #pwmCanvas {
    background-color: #fefefe;
  }
</style>

在本节中，我们将探讨什么是 PWM，以及为什么需要它。

## 数字信号 vs 模拟信号
在数字电路中，信号只有高电平（如 5V 或 3.3V）或低电平（0V）两种状态，没有中间值。这两种截然不同的状态使数字信号非常适合计算机和数字设备，因为它们易于存储、读取和传输，且不会丢失精度。

然而，模拟信号可以在一个范围内连续变化，允许在高电压和低电压之间取任意值。这种平滑的变化对于需要精细控制的应用非常有价值，例如调节音频音量或灯光亮度。

舵机（servo motor）和 LED（用于调光效果）等设备通常需要逐步、精确的电压控制，而模拟信号通过其连续的范围提供了这种能力。

微控制器使用 PWM 来弥合这一差距。

## 什么是 PWM？
PWM 是**脉冲宽度调制（Pulse Width Modulation）**的缩写，它通过快速脉冲式地开关数字信号来产生类似模拟的信号。通过调整脉冲的高电平持续时间，即"占空比（duty cycle）"，可以控制平均输出电压，从而模拟出连续的模拟电平。

 <img style="display: block; margin: auto;" alt="LED PWM" src="../images/led-pwm.jpg" />

## 占空比（Duty Cycle）

**占空比（duty cycle）**是指在一个完整周期内信号处于"开启"状态的时间百分比。

例如：
- 100% 占空比表示信号始终处于开启状态。
- 50% 占空比表示信号一半时间开启，一半时间关闭。
- 0% 占空比表示信号始终处于关闭状态。




这里有一个交互式模拟。使用滑块调整占空比和频率，观察脉冲宽度和 LED 亮度如何变化。方波的上半部分表示信号为高电平（开启）时，下半部分表示信号为低电平（关闭）时。

<canvas id="pwmCanvas" width="800" height="200"></canvas>
<div class="led-container">
  <div class="led-body" id="ledBody"></div>
  <div class="led-pin anode"></div>
  <div class="led-pin cathode"></div>
</div>

<div class="slider-container">
  <label for="dutyCycle">占空比（Duty Cycle）(%): </label>
  <input type="range" id="dutyCycle" min="0" max="100" value="50">
  <span id="dutyCycleValue">50</span>%
</div>
<div class="slider-container">
  <label for="frequency">频率（Frequency）(Hz): </label>
  <input type="range" id="frequency" min="1" max="50" value="10">
  <!-- <span id="frequencyValue">x</span> Hz -->
</div>

如果你在模拟中将占空比从"低到高"再到"高到低"变化，应该会注意到 LED 呈现出一种渐暗的效果。

## 周期与频率
周期（period）是指完成一次完整的开关循环所需的总时间。

PWM 信号的频率（frequency）是指每秒完成的周期数，单位为赫兹（Hz）。频率是周期的倒数。因此，频率越高意味着周期越短，高电平和低电平之间的切换速度越快。

\\[
\\text{频率 (Hz)} = \\frac{1}{\\text{周期 (s)}}
\\]

所以如果周期是 1 秒，那么频率就是 1 Hz。

\\[
1 \\text{Hz} = \\frac{1 \\text{ 个周期}}{1 \\text{ 秒}} = \\frac{1}{1 \\text{ s}}
\\]

例如，如果周期是 20 毫秒（0.02 秒），那么频率就是 50 Hz。

\\[
\\text{频率} = \\frac{1}{20 \\text{ ms}} = \\frac{1}{0.02 \\text{ s}} = 50 \\text{ Hz}
\\]


**根据频率计算每秒的周期数**

计算周期数的公式：
\\[
\\text{周期数} = \\text{频率 (Hz)} \\times \\text{总时间 (秒)}
\\]

如果 PWM 信号的频率为 50 Hz，意味着它每秒完成 50 个周期。

在下一章中，我们将深入探讨 PWM 和定时器。

<script>
  const pwmCanvas = document.getElementById('pwmCanvas');
  const pwmCtx = pwmCanvas.getContext('2d');

  const dutyCycleSlider = document.getElementById('dutyCycle');
  const dutyCycleValue = document.getElementById('dutyCycleValue');
  const frequencySlider = document.getElementById('frequency');
  const frequencyValue = document.getElementById('frequencyValue');
  const ledBody = document.getElementById('ledBody');

  let dutyCycle = 50; // Initial duty cycle in percentage
  let frequency = 10; // Initial frequency in Hz

  function drawPWM() {
    pwmCtx.clearRect(0, 0, pwmCanvas.width, pwmCanvas.height);

    const period = 1000 / frequency; // Period in ms
    const onTime = period * (dutyCycle / 100); // On time in ms
    const offTime = period - onTime; // Off time in ms

    const totalWidth = pwmCanvas.width;
    const cycles = frequency; // Number of cycles to display
    const cycleWidth = totalWidth / cycles;

    pwmCtx.strokeStyle = 'black';
    pwmCtx.lineWidth = 2;
    pwmCtx.beginPath();

    let x = 0;

    if (dutyCycle === 100) {
      pwmCtx.moveTo(0, 50);
      pwmCtx.lineTo(pwmCanvas.width, 50);
    } else if (dutyCycle === 0) {
      pwmCtx.moveTo(0, 150);
      pwmCtx.lineTo(pwmCanvas.width, 150);
    } else {
      for (let i = 0; i < cycles; i++) {
        const highWidth = (onTime / period) * cycleWidth;
        const lowWidth = (offTime / period) * cycleWidth;

        pwmCtx.moveTo(x, 50);
        pwmCtx.lineTo(x + highWidth, 50);
        pwmCtx.lineTo(x + highWidth, 150);
        pwmCtx.lineTo(x + highWidth + lowWidth, 150);
        pwmCtx.lineTo(x + highWidth + lowWidth, 50);

        x += cycleWidth;
      }
    }
    pwmCtx.stroke();
  }

  function updateLED() {
    const brightness = dutyCycle / 100;

    ledBody.style.background = `radial-gradient(circle at center, rgba(255, 85, 85, ${brightness}), #cc0000)`;
  }

  function update() {
    dutyCycle = parseInt(dutyCycleSlider.value, 10);
    frequency = parseInt(frequencySlider.value, 10);

    dutyCycleValue.textContent = dutyCycle;
    // frequencyValue.textContent = frequency;

    drawPWM();
    updateLED();
  }

  dutyCycleSlider.addEventListener('input', update);
  frequencySlider.addEventListener('input', update);

  // Initial draw
  drawPWM();
  updateLED();
</script>

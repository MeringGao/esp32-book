# 深入理解 PWM

<style>
canvas {
    border: 1px solid #000;
    margin-top: 20px;
}
.controls {
    margin: 20px;
}
.control {
    margin: 10px 0;
}
#pwmCanvas {
    background-color: #fefefe;
}
#timerCanvas {
    background-color: #fefefe;
}
#simulation-div{
        background-color: #fefefe;
            text-align: center;

}
</style>

## 定时器的工作原理
定时器（timer）在 PWM 生成器中扮演着关键角色。它从零开始计数到指定的最大值（存储在寄存器中），然后复位并重新开始计数。这个计数过程决定了一个完整周期的持续时间，称为周期（period）。


## 比较值（Compare Value）
定时器的硬件会将其当前计数值与比较值（compare value，存储在寄存器中）进行比较。当计数值小于比较值时，信号保持高电平；当计数值超过比较值时，信号变为低电平。


## PWM 分辨率（Resolution）
在 PWM（脉冲宽度调制）中，分辨率（resolution）指的是占空比（duty cycle）可以被控制的精细程度。这由 PWM 比较寄存器使用的位数决定。

定时器根据分辨率从 0 计数到最大值。分辨率越高，占空比就能被调整得越精细。

对于具有 **n** 位分辨率的系统，定时器可以从 0 计数到 \\(2^n - 1\\)，从而为占空比提供 \\(2^n\\) 个可能的等级。

例如：
- 8 位分辨率允许定时器从 0 计数到 255，提供 256 个可能的占空比等级。
- 10 位分辨率允许定时器从 0 计数到 1023，提供 1024 个可能的占空比等级。

更高的分辨率可以更精确地控制占空比，但也意味着定时器必须在相同的周期内计数更多的值，这可能会降低频率或需要更多的处理能力。本质上，分辨率定义了可以设置的不同占空比值的数量，位数越多，调整越精细。


## 模拟仿真

你可以在这个模拟中修改 PWM 分辨率位数和占空比。调整 PWM 分辨率位数会增加最大计数值，但仍保持在时间周期内（它不会影响占空比）。改变占空比会相应地调整开启和关闭状态，但它也保持在周期内。

  <div class="controls">
    <div class="control">
      <label for="resolution">PWM 分辨率（Resolution）(Bits): </label>
      <input type="range" id="resolution" min="4" max="20" value="8">
      <input type="number" id="resolutionNumber" min="4" max="20" value="8">
    </div>
    <div class="control">
      <label for="dutyCycle">占空比（Duty Cycle）(%): </label>
      <input type="range" id="dutyCycle" min="0" max="100" value="75">
      <input type="number" id="dutyCycleNumber" min="0" max="100" value="75">
    </div>
  </div>
 <div id='simulation-div'>
    <canvas id="timerCanvas" width="800" height="200"></canvas>
    <canvas id="pwmCanvas" width="800" height="150"></canvas>
  </div>

## 占空比、频率与分辨率之间的关系

下图说明了占空比、频率、周期、脉冲宽度和分辨率之间的关系。乍一看可能有点复杂，但分解开来有助于澄清这些概念。

在这个例子中，定时器分辨率为 4 位，意味着定时器从 0 计数到 15。当定时器达到最大值时，会触发溢出中断（overflow interrupt）（蓝色箭头所示），计数器复位为 0。定时器从 0 计数到最大值所需的时间称为"周期（period）"。

<img style="display: block; margin: auto;" alt="PWM" src="../images/pwm-duty-cycle-timer.jpg"/>

占空比设置为 50%，意味着信号在半个周期内保持高电平。在计数过程的每一步，定时器都会将其当前计数值与占空比的比较值进行比较。当定时器计数值超过该比较值时（黄色箭头标记），信号从高电平转变为低电平。这会触发比较中断（compare interrupt），表示状态改变。

信号保持高电平的时间称为脉冲宽度（pulse width）。

  <script>
    const timerCanvas = document.getElementById('timerCanvas');
    const timerCtx = timerCanvas.getContext('2d');

    const pwmCanvas = document.getElementById('pwmCanvas');
    const pwmCtx = pwmCanvas.getContext('2d');

    const resolutionInput = document.getElementById('resolution');
    const resolutionNumber = document.getElementById('resolutionNumber');
    const dutyCycleInput = document.getElementById('dutyCycle');
    const dutyCycleNumber = document.getElementById('dutyCycleNumber');

    const numCountLabels = 1;

    function drawTimerAndPWM(resolution, dutyCycle) {
      const maxTicks = Math.pow(2, resolution) - 1;
      const highTicks = Math.round(maxTicks * (dutyCycle / 100));
      const periodWidth = pwmCanvas.width / 10;

      timerCtx.clearRect(0, 0, timerCanvas.width, timerCanvas.height);
      pwmCtx.clearRect(0, 0, pwmCanvas.width, pwmCanvas.height);

      timerCtx.beginPath();
      const stepsToDraw = Math.min(800, maxTicks);
      const stepIncrement = Math.ceil(maxTicks / stepsToDraw);
      for (let period = 0; period < 10; period++) {
        const startX = period * periodWidth;
        for (let tick = 0; tick <= maxTicks; tick += stepIncrement) {
          const x1 = startX + (tick / maxTicks) * periodWidth;
          const y1 = timerCanvas.height - (tick / maxTicks) * (timerCanvas.height - 100);
          const y2 = timerCanvas.height - ((tick + stepIncrement) / maxTicks) * (timerCanvas.height - 100);
          timerCtx.lineTo(x1, y1);
          if (tick + stepIncrement <= maxTicks) {
            timerCtx.lineTo(x1, y2);
          }
        }
      }
      timerCtx.strokeStyle = 'blue';
      timerCtx.lineWidth = 2;
      timerCtx.stroke();

      timerCtx.font = '12px Arial';
      timerCtx.fillStyle = 'black';
      let labelValues = [];
      if (numCountLabels == 1) {
        labelValues = [0, maxTicks];
      } else if (numCountLabels == 2) {
        labelValues = [0, Math.round(maxTicks / 2), maxTicks];
      } else {
        labelValues = [0, Math.round(maxTicks / 3), Math.round(2 * maxTicks / 3), maxTicks];
      }

      for (let i = 0; i < labelValues.length; i++) {
        const y = timerCanvas.height - (labelValues[i] / maxTicks) * (timerCanvas.height - 100);
        timerCtx.fillText(Math.round(labelValues[i]), 5, y);
      }

      const dutyCycleY = timerCanvas.height - (highTicks / maxTicks) * (timerCanvas.height - 100);
      timerCtx.beginPath();
      timerCtx.moveTo(0, dutyCycleY);
      timerCtx.lineTo(timerCanvas.width, dutyCycleY);
      timerCtx.strokeStyle = 'red';
      timerCtx.lineWidth = 1;
      timerCtx.stroke();

      const maxValueY = timerCanvas.height - (maxTicks / maxTicks) * (timerCanvas.height - 100);
      timerCtx.beginPath();
      timerCtx.moveTo(0, maxValueY);
      timerCtx.lineTo(timerCanvas.width, maxValueY);
      timerCtx.strokeStyle = 'gray';
      timerCtx.lineWidth = 1;
      timerCtx.setLineDash([5, 5]);
      timerCtx.stroke();
      timerCtx.setLineDash([]);

      pwmCtx.beginPath();
      for (let period = 0; period < 10; period++) {
        const startX = period * periodWidth;
        pwmCtx.moveTo(startX, pwmCanvas.height / 2);
        pwmCtx.lineTo(startX + (dutyCycle / 100) * periodWidth, pwmCanvas.height / 2);
        pwmCtx.lineTo(startX + (dutyCycle / 100) * periodWidth, pwmCanvas.height - 20);
        pwmCtx.lineTo(startX + periodWidth, pwmCanvas.height - 20);
        pwmCtx.lineTo(startX + periodWidth, pwmCanvas.height / 2);
      }
      pwmCtx.strokeStyle = 'green';
      pwmCtx.lineWidth = 2;
      pwmCtx.stroke();

      timerCtx.font = '16px Arial';
      timerCtx.fillStyle = 'black';
      timerCtx.fillText(`Resolution: ${resolution} bits`, 10, 20);
      timerCtx.fillText(`Max Timer Count: ${maxTicks}`, 10, 40);

      pwmCtx.font = '16px Arial';
      pwmCtx.fillStyle = 'black';
      pwmCtx.fillText(`Duty Cycle: ${dutyCycle}%`, 10, 20);
    }

    function updateSimulation() {
      const resolution = parseInt(resolutionInput.value);
      const dutyCycle = parseInt(dutyCycleInput.value);
      drawTimerAndPWM(resolution, dutyCycle);
    }

    resolutionInput.addEventListener('input', () => {
      resolutionNumber.value = resolutionInput.value;
      updateSimulation();
    });

    resolutionNumber.addEventListener('input', () => {
      resolutionInput.value = resolutionNumber.value;
      updateSimulation();
    });

    dutyCycleInput.addEventListener('input', () => {
      dutyCycleNumber.value = dutyCycleInput.value;
      updateSimulation();
    });

    dutyCycleNumber.addEventListener('input', () => {
      dutyCycleInput.value = dutyCycleNumber.value;
      updateSimulation();
    });

    updateSimulation();
  </script>

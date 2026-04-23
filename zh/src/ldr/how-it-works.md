## LDR 的工作原理？

我们已经介绍了什么是 LDR。让我再重复一遍：LDR 的阻值会根据照射在它上面的光线量而变化。光线越亮，阻值越低；光线越暗，阻值越高。

德古拉：把 LDR 想象成德古拉。在阳光下，他变得更弱（就像阻值变低一样）。但在黑暗中，他变得更强（就像阻值变高一样）。

我们不会介绍制造 LDR 使用了哪种半导体材料，也不会深入解释它为什么会这样表现。我建议你阅读这篇[文章](https://www.elprocus.com/ldr-light-dependent-resistor-circuit-and-working/)，如果你感兴趣的话可以进一步研究。

### 全亮度下的示例输出
当暴露在完全亮度下时，LDR 的阻值较低，导致输出电压（\\( V_{out} \\)）显著降低。

<img style="display: block; margin: auto;" alt="voltage-divider-ldr1" src="./images/voltage-divider-ldr1.png"/>


### 低光照下的示例输出
光线较少时，LDR 的阻值增加，输出电压也随之增加。

<img style="display: block; margin: auto;" alt="voltage-divider-ldr2" src="./images/voltage-divider-ldr2.png"/>

### 完全黑暗下的示例输出
在黑暗中，LDR 的阻值很高，导致输出电压（\\( V_{out} \\)）更高。

<img style="display: block; margin: auto;" alt="voltage-divider-ldr3" src="./images/voltage-divider-ldr3.png"/>


## 分压器中的 LDR 仿真
你可以调整亮度值并观察 R2（即 LDR）的阻值如何变化。此外，你可以观察当你增加或减少亮度时，\\( V_{out} \\) 电压如何变化。

<style>
canvas {
    border: 1px solid #ccc;
    margin-top: 20px;
    background:white;
}
</style>
<label for="vin">输入电压（Vin）：</label>
<input type="number" id="vin" step="0.01" value="3.3" oninput="updateAndCalculate()"><br><br>
<label for="r1">电阻 R1（Ω）：</label>
<input type="number" id="r1" step="1" value="10000" oninput="updateAndCalculate()"><br><br>
<label for="brightness">亮度：</label>
<input type="range" id="brightness" min="0" max="100" value="50" oninput="updateAndCalculate()">
<span id="brightnessValue">50</span>%<br><br>
<p class="formula" id="formula">
    公式（Formula）：\( V_\text{out} = V_\text{in} \times \frac{R_2}{R_1 + R_2} \)
</p>
<p class="formula" id="filledFormula">
    填充公式（Filled Formula）：V<sub>out</sub> = 3.3 × 999 / (10000 + 999)
</p>
<p id="result">输出电压（Output Voltage，V<sub>out</sub>）：0.25 V</p>
<canvas id="circuitCanvas" width="600" height="400"></canvas>


### Circuitjs
上面的电路图是我使用 Falstad 网站创建的。你可以导入我创建的电路文件 [`voltage-divider-ldr.circuitjs.txt`](./voltage-divider-ldr.circuitjs.txt)，导入到 [Falstad 网站](https://www.falstad.com/circuit/) 中进行实验。
 

<script>
        function updateAndCalculate() {
            updateFormula();
            calculateVoltage();
            drawCircuit();
        }
        function updateFormula() {
            const vin = parseFloat(document.getElementById('vin').value) || 0;
            const r1 = parseFloat(document.getElementById('r1').value) || 0;
            const brightness = parseFloat(document.getElementById('brightness').value);
            const rMin = 999;
            const rMax = 100000;
            const r2 = rMin + (rMax - rMin) * (100 - brightness) / 100;
            document.getElementById('filledFormula').innerHTML = 
                `填充公式（Filled Formula）：V<sub>out</sub> = ${vin.toFixed(2)} × ${r2.toFixed(2)} / (${r1} + ${r2.toFixed(2)})`;
        }
        function calculateVoltage() {
            const vin = parseFloat(document.getElementById('vin').value);
            const r1 = parseFloat(document.getElementById('r1').value);
            const brightness = parseFloat(document.getElementById('brightness').value);
            const rMin = 999;
            const rMax = 100000;
            const r2 = rMin + (rMax - rMin) * (100 - brightness) / 100;
            document.getElementById('brightnessValue').textContent = brightness;
            if (isNaN(vin) || isNaN(r1) || r1 <= 0 || r2 <= 0) {
                document.getElementById('result').textContent = 
                    "Please enter valid positive numbers for all fields.";
                return;
            }
            const vout = vin * (r2 / (r1 + r2));
            document.getElementById('result').innerHTML = 
                `输出电压（Output Voltage，V<sub>out</sub>）：${vout.toFixed(2)} V`;
        }
        function drawZigZagResistor(ctx, x, y, width, height) {
            const segments = 6;
            const step = height / segments;
            const segmentWidth = width / 2;
            ctx.beginPath();
            ctx.moveTo(x, y);
            for (let i = 0; i < segments; i++) {
                const offset = (i % 2 === 0) ? segmentWidth : -segmentWidth;
                ctx.lineTo(x + offset, y + step * (i + 1));
            }
            const finalOffset = (segments % 2 === 0) ? segmentWidth : -segmentWidth;
            ctx.lineTo(x + 1, y + height + 4);
            ctx.lineWidth = 2;
            ctx.stroke();
        }
        function drawArrow(ctx, x, y, length, angle) {
            ctx.beginPath();
            ctx.moveTo(x, y);
            ctx.lineTo(x + length * Math.cos(angle), y + length * Math.sin(angle));
            ctx.lineWidth = 1;
            ctx.stroke();
            const arrowSize = 11;
            const angleLeft = angle + Math.PI / 6;
            const angleRight = angle - Math.PI / 6;
            ctx.beginPath();
            ctx.moveTo(x + length * Math.cos(angle), y + length * Math.sin(angle));
            ctx.lineTo(x + (length - arrowSize) * Math.cos(angleLeft), y + (length - arrowSize) * Math.sin(angleLeft));
            ctx.moveTo(x + length * Math.cos(angle), y + length * Math.sin(angle));
            ctx.lineTo(x + (length - arrowSize) * Math.cos(angleRight), y + (length - arrowSize) * Math.sin(angleRight));
            ctx.stroke();
        }
        function drawGroundSymbol(ctx, x, y) {
            const lineSpacing = 5;
            const topLineWidth = 20;
            ctx.beginPath();
            ctx.moveTo(x - topLineWidth / 2, y);
            ctx.lineTo(x + topLineWidth / 2, y);
            ctx.stroke();
            const middleLineWidth = topLineWidth * 0.6;
            ctx.beginPath();
            ctx.moveTo(x - middleLineWidth / 2, y + lineSpacing);
            ctx.lineTo(x + middleLineWidth / 2, y + lineSpacing);
            ctx.stroke();
            const bottomLineWidth = middleLineWidth * 0.6;
            ctx.beginPath();
            ctx.moveTo(x - bottomLineWidth / 2, y + 2 * lineSpacing);
            ctx.lineTo(x + bottomLineWidth / 2, y + 2 * lineSpacing);
            ctx.stroke();
        }
        function formatResistance(resistance) {
            return resistance >= 1000
                ? `${(resistance / 1000).toFixed(1)} kΩ`
                : `${resistance.toFixed(0)} Ω`;
        }
        function drawCircuit() {
            const canvas = document.getElementById('circuitCanvas');
            const ctx = canvas.getContext('2d');
            const vin = parseFloat(document.getElementById('vin').value) || 0;
            const r1 = parseFloat(document.getElementById('r1').value) || 0;
            const brightness = parseFloat(document.getElementById('brightness').value);

            const rMin = 999;
            const rMax = 100000;

            const r2 = rMin + (rMax - rMin) * (100 - brightness) / 100;

            const vout = (r1 + r2) !== 0 ? (vin * (r2 / (r1 + r2))) : 0;

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            ctx.fillStyle = 'black';
            ctx.font = '14px Arial';

            ctx.fillText(`V_in: ${vin.toFixed(2)} V`, 38, 96);
            ctx.beginPath();
            ctx.moveTo(100, 100);
            ctx.lineTo(150, 100);
            ctx.stroke();

            ctx.moveTo(130, 100);
            ctx.lineTo(170, 100);
            ctx.stroke();

            ctx.beginPath();
            ctx.moveTo(170, 100);
            ctx.lineTo(170, 130);
            ctx.stroke();

            ctx.fillText(`R1: ${formatResistance(r1)}`, 200, 155);
            drawZigZagResistor(ctx, 170, 130, 23, 40);

            ctx.beginPath();
            ctx.moveTo(170, 175);
            ctx.lineTo(170, 230);
            ctx.stroke();

            ctx.fillText(`R2: ${formatResistance(r2)}`, 200, 256);
            drawZigZagResistor(ctx, 170, 230, 20, 40);

            const arrowX = 170;
            const arrowY = 220;
            const angle = Math.PI / 4;

            drawArrow(ctx, arrowX - 30, arrowY, 25, angle);
            drawArrow(ctx, arrowX - 33, arrowY + 22, 25, angle);

            ctx.beginPath();
            ctx.moveTo(170, 275);
            ctx.lineTo(170, 300);
            ctx.stroke();
            ctx.fillText('地线（Ground）', 160, 330);
            
            drawGroundSymbol(ctx, 170, 300);

            ctx.beginPath();
            ctx.moveTo(170, 200);
            ctx.lineTo(270, 200);
            ctx.stroke();
            ctx.fillText(`V_out: ${vout.toFixed(2)} V`, 280, 203);
        }

        updateAndCalculate();
    </script>

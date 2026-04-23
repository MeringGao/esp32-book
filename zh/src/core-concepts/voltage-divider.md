# 分压器（Voltage Divider）

**分压器（voltage divider）**是一种简单的电路，使用两个串联电阻将输入电压 \\( V_{in} \\) 降低到较低的输出电压 \\( V_{out} \\)。连接到输入电压 \\( V_{in} \\) 的电阻称为 \\( R_{1} \\)，另一个电阻称为 \\( R_{2} \\)。输出电压 \\( V_{out} \\) 从 \\( R_{1} \\) 和 \\( R_{2} \\) 之间的连接点取出，产生 \\( V_{in} \\) 的一部分。

## 电路

<img style="display: block; margin: auto;" alt="Voltage Divider" src="./images/voltage-divider.png"/>

输出电压（V<sub>out</sub>）使用以下公式计算：

\\[
V_{out} = V_{in} \\times \\frac{R_2}{R_1 + R_2}
\\]

### 计算 \\( V_{out} \\) 的示例

已知：
- \\( V_{in} = 3.3V \\)
- \\( R_1 = 10 k\\Omega \\)
- \\( R_2 = 10 k\\Omega \\)

代入数值：

\\[
V_{out} = 3.3V \\times \\frac{10 k\\Omega}{10 k\\Omega + 10 k\\Omega} = 3.3V \\times \\frac{10}{20} = 3.3V \\times 0.5 = 1.65V
\\]


输出电压 \\( V_{out} \\) 为 1.65V。


```rust,editable
fn main() {
    // 你可以编辑这段代码
    // You can edit the code
    // 你可以修改数值并运行代码
    // You can modify values and run the code
    let vin: f64 = 3.3;
    let r1: f64 = 10000.0;
    let r2: f64 = 10000.0;

    let vout = vin * (r2 / (r1 + r2));

    println!("The output voltage Vout is: {:.2} V", vout);
}
```

## 应用场景

分压器用于电位器（potentiometer）等应用中，当旋钮旋转时电阻发生变化，从而调整输出电压。它们还用于测量电阻式传感器，如光敏传感器和热敏电阻（thermistor），其中施加已知电压，微控制器读取中心节点的电压以确定温度等传感器值。

## 分压器模拟
<style>
canvas {
    border: 1px solid #ccc;
    margin-top: 20px;
    background:white;
}
</style>
<label for="vin">输入电压（Input Voltage）(V<sub>in</sub>):</label>
<input type="number" id="vin" step="0.01" value="3.3" oninput="updateAndCalculate()"><br><br>

<label for="r1">电阻 R1 (Ω):</label>
<input type="number" id="r1" step="1" value="10000" oninput="updateAndCalculate()"><br><br>

<label for="r2">电阻 R2 (Ω):</label>
<input type="number" id="r2" step="1" value="10000" oninput="updateAndCalculate()"><br><br>

<p class="formula" id="formula">
    公式（Formula）: V<sub>out</sub> = V<sub>in</sub> × (R<sub>2</sub> / (R<sub>1</sub> + R<sub>2</sub>))
</p>
<p class="formula" id="filledFormula">
    代入数值（Filled Formula）: V<sub>out</sub> = 3.3 × (10000 / (10000 + 10000))
</p>
<p id="result">输出电压（Output Voltage）(Vout): 1.65 V</p>

<canvas id="circuitCanvas" width="600" height="400"></canvas>


## Falstad 网站上的模拟器

我使用网站 [https://www.falstad.com/circuit/](https://www.falstad.com/circuit/) 创建了该电路图。这是一个绘制电路的好工具。你可以下载我创建的文件 [`voltage-divider.circuitjs.txt`](./voltage-divider.circuitjs.txt)，并导入以进行电路实验。


<script>
    function updateAndCalculate() {
        updateFormula();
        calculateVoltage();
        drawCircuit();
    }

    function updateFormula() {
        const vin = parseFloat(document.getElementById('vin').value) || 0;
        const r1 = parseFloat(document.getElementById('r1').value) || 0;
        const r2 = parseFloat(document.getElementById('r2').value) || 0;

        document.getElementById('filledFormula').textContent =
            `代入数值（Filled Formula）: Vout = ${vin} × (${r2} / (${r1} + ${r2}))`;
    }

    function calculateVoltage() {
        const vin = parseFloat(document.getElementById('vin').value);
        const r1 = parseFloat(document.getElementById('r1').value);
        const r2 = parseFloat(document.getElementById('r2').value);

        if (isNaN(vin) || isNaN(r1) || isNaN(r2) || r1 <= 0 || r2 <= 0) {
            document.getElementById('result').textContent =
                "请输入所有字段的有效正数（Please enter valid positive numbers for all fields）.";
            return;
        }

        const vout = vin * (r2 / (r1 + r2));
        document.getElementById('result').textContent =
            `输出电压（Output Voltage）(Vout): ${vout.toFixed(2)} V`;
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

    function drawCircuit() {
        const canvas = document.getElementById('circuitCanvas');
        const ctx = canvas.getContext('2d');
        const vin = parseFloat(document.getElementById('vin').value) || 0;
        const r1 = parseFloat(document.getElementById('r1').value) || 0;
        const r2 = parseFloat(document.getElementById('r2').value) || 0;
        const vout = vin * (r2 / (r1 + r2)) || 0;

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = 'black';
        ctx.font = '16px Arial';
        ctx.fillText(`V_in: ${vin.toFixed(2)} V`, 35, 94);
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

        ctx.fillText(`R1: ${r1} Ω`, 200, 155);
        drawZigZagResistor(ctx, 170, 130, 23, 40);

        ctx.beginPath();
        ctx.moveTo(170, 175);
        ctx.lineTo(170, 230);
        ctx.stroke();

        ctx.fillText(`R2: ${r2} Ω`, 200, 256);
        drawZigZagResistor(ctx, 170, 230, 20, 40);

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

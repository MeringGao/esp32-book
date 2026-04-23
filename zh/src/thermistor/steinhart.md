
## Steinhart-Hart 方程

Steinhart-Hart 方程在更宽的温度范围内提供了更精确的温度-阻值关系。

\\[
\frac{1}{T} = A + B \ln R + C (\ln R)^3
\\]

其中：
- T 是 **开尔文（Kelvins）** 为单位的温度。（从摄氏度计算开尔文的公式：K = °C + 273.15）
- R 是温度 T 时的阻值，单位为 **欧姆（Ohms）**。
- A、B 和 C 是热敏电阻材料特有的常数，通常由制造商提供。为了获得更好的精度，你可能需要自行校准并确定这些值。一些数据手册提供了各种温度下的阻值，也可以用来计算这些常数。

**注意**：

我们不会在本练习中使用这个方程，因为找到 A、B 和 C 常数需要更多的工作。B 方程对我们的目的来说已经足够精确，所以如果你愿意，可以跳过这一章。


### 校准

为了确定 A、B 和 C 的精确值，将热敏电阻置于三种温度条件下：室温、冰水和沸水。对于每种条件，使用 ADC 值测量热敏电阻的阻值，并使用可靠的温度计记录实际温度。使用阻值和对应的温度，计算系数：
- 将 A 分配给冰水温度，
- 将 B 分配给室温，
- 将 C 分配给沸水温度。

### 计算 Steinhart-Hart 系数

有了三个阻值和温度数据点，我们可以求出 A、B 和 C。

$$
\begin{bmatrix}
    1 & \ln R_1 & \ln^3 R_1 \\
    1 & \ln R_2 & \ln^3 R_2 \\
    1 & \ln R_3 & \ln^3 R_3
\end{bmatrix}\begin{bmatrix}
    A \\
    B \\
    C
\end{bmatrix} = \begin{bmatrix}
    \frac{1}{T_1} \\
    \frac{1}{T_2} \\
    \frac{1}{T_3}
\end{bmatrix}
$$

其中：
- \( R_1, R_2, R_3 \) 是温度 \( T_1, T_2, T_3 \) 时的阻值。

**让我们计算这些系数**

计算阻值的自然对数：
$$
L_1 = \ln R_1, \quad L_2 = \ln R_2, \quad L_3 = \ln R_3
$$

中间计算：
$$
Y_1 = \frac{1}{T_1}, \quad Y_2 = \frac{1}{T_2}, \quad Y_3 = \frac{1}{T_3}
$$

$$
\gamma_2 = \frac{Y_2 - Y_1}{L_2 - L_1}, \quad \gamma_3 = \frac{Y_3 - Y_1}{L_3 - L_1}
$$

所以，最终：
$$
C = \left( \frac{ \gamma_3 - \gamma_2 }{ L_3 - L_2} \right) \left(L_1 + L_2 + L_3\right)^{-1} \\
$$
$$
B = \gamma_2 - C \left(L_1^2 + L_1 L_2 + L_2^2\right) \\
$$
$$
A = Y_1 - \left(B + L_1^2 C\right) L_1
$$


<span style="color: green;">好消息，各位！</span> 你不需要手动计算这些系数。只需提供冷、室温和热环境下的阻值和温度值，然后使用下面的表单来确定 A、B 和 C。

### ADC 值和阻值计算
<span style="color: orange;">注意：</span> 如果你已经有了温度和对应的阻值，可以直接使用第二个表格输入这些值。

如果你有 ADC 值并想计算阻值，请使用此表格来查找不同温度下的对应阻值。当你输入每个温度下的 ADC 值时，计算出的阻值将自动更新到第二个表格中。

要执行此计算，你需要热敏电阻的基准阻值，这对于根据 ADC 值确定给定温度下的阻值至关重要。

请注意，如果你使用的是不同的微控制器，可能需要调整 ADC 位数。在我们的例子中，对于 ESP32，ADC 分辨率（resolution）为 12 位。

<form id="adcForm">
  <label for="baseResistance">基准阻值 (Ω): </label>
  <input type="number" id="baseResistance" name="baseResistance" step="any" value="10000" oninput="updateResistance()">
    <br/>
    <label for="adcBits">ADC 位数: </label>
  <input type="number" id="adcBits" name="adcBits" step="any" value="12" oninput="updateResistance()">
  <br/>
  <br/>

  <table>
    <thead>
      <tr>
         <th>环境</th>
        <th>ADC 值</th>
      </tr>
    </thead>
    <tbody>
      <!-- Cold Water Row -->
      <tr>
        <td>冷水</td>
        <td><input type="number" id="adcColdCount" name="adcColdCount" step="any" oninput="updateResistance()"></td>
      </tr>
      <!-- Room Temperature Row -->
      <tr>
        <td>室温</td>
        <td><input type="number" id="adcRoomCount" name="adcRoomCount" step="any" oninput="updateResistance()"></td>
      </tr>
      <!-- Boiling Water Row -->
      <tr>
      <td>沸水</td>
        <td><input type="number" id="adcBoilCount" name="adcBoilCount" step="any" oninput="updateResistance()"></td>
      </tr>
    </tbody>
  </table>
</form>


### 系数查找器

通过在华氏度或摄氏度中输入值来调整温度；表单将自动转换为另一种格式。提供每个温度对应的阻值，然后点击 "Calculate Coefficients" 按钮。

<form id="steinhartForm" onsubmit="calcCoeffBtnClicked(event)">
<table>
<thead>
<tr>
<th>环境</th>
<th>阻值 (欧姆)</th>
<th>温度 (°F)</th>
<th>温度 (°C)</th>
<th>温度 (K)</th>
</tr>
</thead>
<tbody>
<tr>
<td>冷水</td>
<td><input type="number" id="resistanceCold" name="resistanceCold" step="any" oninput="validateInput()"></td>
<td><input type="number" id="coldTempF" name="coldTempF" step="any" oninput="calcTempFromFarenhit('coldTempC', 'coldTempF', 'coldTempK', 'resistanceCold')"></td>
<td><input type="number" id="coldTempC" name="coldTempC" step="any" oninput="calcTempFromCel('coldTempC', 'coldTempF', 'coldTempK', 'resistanceCold')"></td>

<td><input type="number" id="coldTempK" name="coldTempK" step="any" readonly></td>
</tr>
<tr>
<td>室温</td>
<td><input type="number" id="resistanceRoom" name="resistanceRoom" step="any" oninput="validateInput()"></td>
<td><input type="number" id="roomTempF" name="roomTempF" step="any" oninput="calcTempFromFarenhit('roomTempC', 'roomTempF', 'roomTempK', 'resistanceRoom')"></td>
<td><input type="number" id="roomTempC" name="roomTempC" step="any" value="25"  oninput="calcTempFromCel('roomTempC', 'roomTempF', 'roomTempK', 'resistanceRoom')"></td>
<td><input type="number" id="roomTempK" name="roomTempK" step="any" readonly></td>
</tr>
<tr>
<td>沸水</td>
<td><input type="number" id="resistanceBoiling" name="resistanceBoiling"  step="any" oninput="validateInput()"></td>

<td><input type="number" id="boilTempF" name="boilTempF" step="any"  oninput="calcTempFromFarenhit('boilTempC', 'boilTempF', 'boilTempK', 'resistanceBoiling')"></td>
<td><input type="number" id="boilTempC" name="boilTempC" step="any" oninput="calcTempFromCel('boilTempC', 'boilTempF', 'boilTempK', 'resistanceBoiling')"></td>
<td><input type="number" id="boilTempK" name="boilTempK" step="any" readonly></td>
</tr>
</tbody>
</table>

<h3>结果</h3>
<p>
    A: <input type="text" id="resultA" readonly />
    <span id="actualA"></span> 
</p>
<p>
    B: <input type="text" id="resultB" readonly />
    <span id="actualB"></span> 
</p>
<p>
    C: <input type="text" id="resultC" readonly />
    <span id="actualC"></span> 
</p>


<button type="submit" id="submitBtn" >计算系数</button>
</form>

<h3>从阻值计算温度</h3>
<p>现在，有了这些系数，你可以计算任何给定阻值的温度：</p>

<label for="r">R (欧姆): </label>
<input type="number" name="r" value="10000" id="inputResistance" >

<button type="button" id="calculateBtn" onclick="calculateTemperatureFromResistance()" >计算温度</button>

<label for="tc">结果 (°C): </label>
<input type="text" name="tc" id="resultCelsius" readonly>

<label for="tf">结果 (°F): </label>
<input type="text" name="tf" id="resultFahrenheit" readonly>


<!-- Error Message Section -->
<p id="errorMessage" style="color: red; display: none;">错误：请先计算系数 (A, B, C)。</p>

<script>
window.onload = function() {
  // 冷水默认值
  document.getElementById("resistanceCold").value = 25000;
  document.getElementById("coldTempC").value = 5;
  calcTempFromCel('coldTempC', 'coldTempF', 'coldTempK', 'resistanceCold');
  
  // 室温默认值
  document.getElementById("resistanceRoom").value = 10000;
  document.getElementById("roomTempC").value = 25;
  calcTempFromCel('roomTempC', 'roomTempF', 'roomTempK', 'resistanceRoom');
  
  // 沸水默认值
  document.getElementById("resistanceBoiling").value = 4000;
  document.getElementById("boilTempC").value = 45;
  calcTempFromCel('boilTempC', 'boilTempF', 'boilTempK', 'resistanceBoiling');

  calculateCoefficients();
};

// Function to calculate resistance based on base resistance and ADC value
function calculateResistance(baseResistance, adcCount, adcBits) {
  const maxADCValue = Math.pow(2, adcBits) - 1;  // Max ADC value for the given bits (e.g., 12 bits = 4095)
  
  const resistance = baseResistance * ((maxADCValue / adcCount)-1);
  
  return resistance;
}

function updateResistance() {
  const baseResistance = parseFloat(document.getElementById("baseResistance").value);
  const adcBits = parseInt(document.getElementById("adcBits").value);
  
  const adcColdCount = parseFloat(document.getElementById("adcColdCount").value);
  const adcRoomCount = parseFloat(document.getElementById("adcRoomCount").value);
  const adcBoilCount = parseFloat(document.getElementById("adcBoilCount").value);
  
  // Calculate resistance for each environment using the ADC counts
  if (!isNaN(baseResistance) && !isNaN(adcBits)) {
    const resistanceCold = calculateResistance(baseResistance, adcColdCount, adcBits);
    document.getElementById("resistanceCold").value = resistanceCold.toFixed(2);

    const resistanceRoom = calculateResistance(baseResistance, adcRoomCount, adcBits);
    document.getElementById("resistanceRoom").value = resistanceRoom.toFixed(2);

    const resistanceBoiling = calculateResistance(baseResistance, adcBoilCount, adcBits);
    document.getElementById("resistanceBoiling").value = resistanceBoiling.toFixed(2);
  }
}

function calcTempFromCel(celsiusId, fahrenheitId, kelvinId, resistanceId) {
    const tempC = parseFloat(document.getElementById(celsiusId).value);

    if (!isNaN(tempC)) {
        const tempF = (tempC * 9/5) + 32;
        const tempK = tempC + 273.15;
        document.getElementById(fahrenheitId).value = tempF.toFixed(2);
        document.getElementById(kelvinId).value = tempK.toFixed(2);
    } else{
        document.getElementById(fahrenheitId).value = "";
        document.getElementById(kelvinId).value = "";
    }
}

function calcTempFromFarenhit(celsiusId, fahrenheitId, kelvinId, resistanceId) {
    const tempF = parseFloat(document.getElementById(fahrenheitId).value);
    if (!isNaN(tempF)) {
        const tempC = (tempF - 32) * 5 / 9;
        const tempK = tempC + 273.15;
        document.getElementById(celsiusId).value = tempC.toFixed(2);
        document.getElementById(kelvinId).value = tempK.toFixed(2);
    } else{
        document.getElementById(celsiusId).value = "";
        document.getElementById(kelvinId).value = "";
    }
}


function validateInput() {
    const resistanceCold = document.getElementById("resistanceCold").value;
    const resistanceRoom = document.getElementById("resistanceRoom").value;
    const resistanceBoiling = document.getElementById("resistanceBoiling").value;
    const coldTempC = document.getElementById("coldTempC").value;
    const roomTempC = document.getElementById("roomTempC").value;
    const boilTempC = document.getElementById("boilTempC").value;
    const submitBtn = document.getElementById("submitBtn");
}

function calcCoeffBtnClicked(event){
    event.preventDefault();
    calculateCoefficients();
}

function calculateCoefficients() {
    const T1 = parseFloat(document.getElementById("coldTempK").value);
    const T2 = parseFloat(document.getElementById("roomTempK").value);
    const T3 = parseFloat(document.getElementById("boilTempK").value);

    const resistanceCold = parseFloat(document.getElementById("resistanceCold").value);
    const resistanceRoom = parseFloat(document.getElementById("resistanceRoom").value);
    const resistanceBoiling = parseFloat(document.getElementById("resistanceBoiling").value);

    const L1 = Math.log(resistanceCold); //natural logarithm
    const L2 = Math.log(resistanceRoom);
    const L3 = Math.log(resistanceBoiling);

    const Y1 = 1 / T1;
    const Y2 = 1 / T2;
    const Y3 = 1 / T3;

    const gamma2 = (Y2 - Y1) / (L2 - L1); //γ2
    const gamma3 = (Y3 - Y1) / (L3 - L1); //γ3

    // Calculate coefficients A, B, and C
    const C = ((gamma3 - gamma2) / (L3 - L2)) * (L1 + L2 + L3) ** -1;
    const B = gamma2 - C * (Math.pow(L1, 2) + L1 * L2 + Math.pow(L2, 2));
    const A = Y1 - (B + Math.pow(L1, 2) * C) * L1;

    document.getElementById("resultA").value = A.toExponential(8);
    document.getElementById("resultB").value = B.toExponential(8);
    document.getElementById("resultC").value = C.toExponential(8);

    document.getElementById("actualA").textContent = `(${A.toFixed(16)})`;
    document.getElementById("actualB").textContent = `(${B.toFixed(16)})`;
    document.getElementById("actualC").textContent = `(${C.toFixed(16)})`;
}

function calculateTemperatureFromResistance() {

    const A = parseFloat(document.getElementById('resultA').value);
    const B = parseFloat(document.getElementById('resultB').value);
    const C = parseFloat(document.getElementById('resultC').value);

    if (isNaN(A) || isNaN(B) || isNaN(C)) {
        document.getElementById('errorMessage').style.display = 'block'; 
        document.getElementById('resultFahrenheit').value = '';
        document.getElementById('resultCelsius').value = '';
        return;
    } else{
            document.getElementById('errorMessage').style.display = 'none';
    }

    let resistance = parseFloat(document.getElementById('inputResistance').value);
    if (isNaN(resistance)) {
        alert("请输入有效的阻值。");
        return;
    }

    // Calculate temperature in Kelvin using Steinhart-Hart equation: 
    // 1/T = A + B*ln(R) + C*(ln(R))^3
    let inverseTemperature = A + B * Math.log(resistance) + C * Math.pow(Math.log(resistance), 3);
    let temperatureKelvin = 1 / inverseTemperature; 

    // Convert to Celsius and Fahrenheit
    let temperatureCelsius = temperatureKelvin - 273.15;  
    let temperatureFahrenheit = (temperatureCelsius * 9/5) + 32;  

    document.getElementById('resultFahrenheit').value = temperatureFahrenheit.toFixed(2);
    document.getElementById('resultCelsius').value = temperatureCelsius.toFixed(2);

}
</script>

### Rust 函数
```rust
fn steinhart_temp_calc(
    resistance: f64, // 阻值，单位为欧姆
    a: f64,          // 系数 A
    b: f64,          // 系数 B
    c: f64,          // 系数 C
) -> Result<(f64, f64), String> {
    if resistance <= 0.0 {
        return Err("阻值必须是正数。".to_string());
    }

    // 使用 Steinhart-Hart 方程计算开尔文温度：
    // 1/T = A + B*ln(R) + C*(ln(R))^3
    let ln_r = resistance.ln();
    let inverse_temperature = a + b * ln_r + c * ln_r.powi(3);

    if inverse_temperature == 0.0 {
        return Err("无效的系数或阻值导致除零。".to_string());
    }

    let temperature_kelvin = 1.0 / inverse_temperature;

    let temperature_celsius = temperature_kelvin - 273.15;
    let temperature_fahrenheit = (temperature_celsius * 9.0 / 5.0) + 32.0;

    Ok((temperature_celsius, temperature_fahrenheit))
}

fn main() {
    // 示例输入
     let a = 2.10850817e-3;
    let b = 7.97920473e-5;
    let c = 6.53507631e-7;
    let resistance = 10000.0;


    match steinhart_temp_calc(resistance, a, b, c) {
        Ok((celsius, fahrenheit)) => {
            println!("摄氏温度: {:.2}", celsius);
            println!("华氏温度: {:.2}", fahrenheit);
        }
        Err(e) => println!("错误: {}", e),
    }
}
```

### 参考资料
- [Thermistor Calculator](https://www.thinksrs.com/downloads/programs/therm%20calc/ntccalibrator/ntccalculator.html) 
- [Thermistor Steinhart-Hart Coefficients for Calculating Motor Temperature](https://www.servo.jp/member/admin/document_upload/AN144-Thermistor-Steinhart-Hart-Coefficients.pdf) 
- [Calibrate Steinhart-Hart Coefficients for Thermistors](https://www.thinksrs.com/downloads/PDFs/ApplicationNotes/LDC%20Note%204%20NTC%20Calculatorold.pdf) 
- [Cooking Thermometer With Steinhart-Hart Correction](https://www.instructables.com/ESP32-NTP-Temperature-Probe-Cooking-Thermometer-Wi/)

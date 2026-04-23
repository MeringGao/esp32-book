
## B 方程

B 方程更简单，但精度较低。

\\[
\frac{1}{T} = \frac{1}{T_0} + \frac{1}{B} \ln \left( \frac{R}{R_0} \right)
\\]

其中：

- T 是 **开尔文（Kelvin）** 为单位的温度。
- \( T_0 \) 是参考温度（通常为 298.15K 或 25°C），在此温度下热敏电阻的阻值是已知的（通常为 10kΩ）。
- R 是温度 T 时的 **阻值**。
- \( R_0 \) 是参考温度 \( T_0 \) 时的 **阻值**（通常为 10kΩ）。
- B 是热敏电阻的 **B 值**。

B 值是一个通常由制造商提供的常数，根据热敏电阻的材料而变化。它描述了在两个点之间的特定温度范围内电阻曲线的斜率（即 \( T_0 \) 对 \( R_0 \) 和 T 对 R）。你甚至可以通过校准两个温度下的阻值来自己重写上述公式以求得 B 值。

**示例计算：**

已知：
- 参考温度 \( T_0 = 298.15K \)（即 25°C + 273.15 转换为开尔文）
- 参考阻值 \( R_0 = 10k\Omega \)
- B 值 B = 3950（许多热敏电阻的典型值）
- 温度 T 时测得的阻值：10475Ω

### 步骤 1：应用 B 参数方程

代入给定值：

\\[
\frac{1}{T} = \frac{1}{298.15} + \frac{1}{3950} \ln \left( \frac{10,475}{10,000} \right)
\\]

\\[
\frac{1}{T} = 0.003354016 + \frac{1}{3950} \ln(1.0475)
\\]

\\[
\frac{1}{T} = 0.003354016 + (0.000011748)
\\]

\\[
\frac{1}{T} = 0.003365764
\\]

### 步骤 2：计算温度 (T)

\\[
T = \frac{1}{0.003365764} = 297.10936358 (开尔文)
\\]

转换为摄氏度：

\\[
T_{Celsius} = 297.10936358 - 273.15 \approx 23.95936358°C
\\]

### 结果：

对应于 10475Ω 阻值的温度约为 **23.96°C**。

### Rust 函数

```rust
const fn kelvin_to_celsius(kelvin: f64) -> f64 {
    kelvin - 273.15
}

const fn celsius_to_kelvin(celsius: f64) -> f64 {
    celsius + 273.15
}

const REF_RES: f64 = 10_000.0; // 参考阻值，单位为欧姆（10kΩ）
const REF_TEMP: f64 = 25.0;  // 参考温度 25°C
const REF_TEMP_K: f64 = celsius_to_kelvin(REF_TEMP); // T0

fn calculate_temperature(current_res: f64, b_val: f64) -> f64 {
    let ln_value = (current_res/REF_RES).ln();
    // let ln_value = libm::log(current_res / ref_res); // 在 no_std 中使用此 crate
    let inv_t = (1.0 / REF_TEMP_K) + ((1.0 / b_val) * ln_value);
    1.0 / inv_t
}


const B_VALUE: f64 = 3950.0;
const V_IN: f64 = 3.3; // 输入电压

fn main() {
    let r = 9546.0; // 测得的阻值，单位为欧姆
    
    let temperature_kelvin = calculate_temperature(r, B_VALUE);
    let temperature_celsius = kelvin_to_celsius(temperature_kelvin);
    println!("Temperature: {:.2} °C", temperature_celsius);
}

```

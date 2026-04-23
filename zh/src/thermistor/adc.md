
# 从 ADC 计算阻值

当使用 ESP32 设置热敏电阻时，我们无法直接获得电压值。相反，我们收到的是一个 ADC 值（请参阅 [ADC](../core-concepts/adc/index.md) 章节）。我们需要从 ADC 值中获取阻值，以便进行热敏电阻的温度计算（这将在接下来的章节中讨论）。

我们将使用以下公式从 ADC 读数计算阻值。如果你需要了解其推导过程，请参阅 [从 ADC 值推导阻值](./adc-maths.md)。

\\[
R_2 = \frac{R_1}{\left( \frac{\text{ADC_MAX}}{\text{adc_value}} - 1 \right)}
\\]

也可以写成这样：

\\[
R_2 = R_1 \left( \frac{\text{adc_value}}{\text{ADC_MAX} - \text{adc_value}} \right)
\\]

其中：
- **R2**：我们需要计算的热敏电阻阻值
- **R1**：与热敏电阻串联的电阻值（通常为 10kΩ）
- **ADC_MAX**：最大 ADC 值，对于 12 位 ADC 为 4095（\( 2^{12} - 1 \)）
- **adc_value**：从 ADC 读取的值

### Rust 函数

```rust

const ADC_MAX: f64 = 4095.0;
const R1_RES: f64 = 10_000.0;

const fn adc_to_resistance(adc_value: f64) -> f64 {
    let x: f64 = adc_value/(ADC_MAX - adc_value);
    R1_RES * x
}

fn main() {
    let adc_value = 2000; // 我们的示例 ADC 值

    let r2 = adc_to_resistance(adc_value as f64);
    println!("计算得到的阻值 (R2): {} Ω", r2);
}
```

<br/>

**注意：**

如果你将热敏电阻连接到了电源而不是 GND，那么你需要使用相反的公式，因为此时热敏电阻变成了 R1。

\\[
R_1 = {R_2} \times \left(\frac{\text{ADC_MAX}}{\text{adc_value}} - 1\right)
\\]

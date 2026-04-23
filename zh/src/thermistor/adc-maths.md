# 从 ADC 值推导阻值

> [!CAUTION]
> 本节中展示的数学推导目前正在审查中。有用户在 [GitHub issue](https://github.com/ImplFerris/esp32-book/issues/16) 中质疑这种方法的准确性。看来我最初的研究和推导是错误的。我尚未进一步研究并验证这些公式，分析完成后将更新本节。在此之前，请不要依赖此公式。

如果你愿意，可以跳过这一章。它只是解释了从 ADC 值推导阻值背后的数学原理。我们结合分压器公式和 ADC 分辨率公式来求出阻值 R2。

<span style="color:orange">注意：</span> 这里假设热敏电阻的一侧连接到地（GND）。我注意到有些在线文章做法相反，将热敏电阻的一侧连接到电源，这最初让我有些困惑。当热敏电阻的一侧连接到 GND，另一侧连接到 ADC 引脚时，它在分压器中就是 R2。

**分压器公式**
\\[
V_{out} = V_{in} \times \frac{R_2}{R_1 + R_2}
\\]

## 步骤 1：结合公式

要根据 ADC 原始结果计算电压，可以使用以下公式：

\\[
V_{out} = V_{in} \times \frac{\text{adc_value}}{\text{ADC_MAX}}
\\]

通过将这个关系代入分压器公式：

\\[
V_{in} \times \frac{\text{adc_value}}{\text{ADC_MAX}} = V_{in} \times \frac{R_2}{R_1 + R_2}
\\]

---

## 步骤 2：消去 Vin

我们可以从等式两边消去 Vin。

\\[
\require{cancel}
\cancel{V_{in}} \times \frac{\text{adc_value}}{\text{ADC_MAX}} = \cancel{V_{in}} \times \frac{R_2}{R_1 + R_2}
\\]

这简化为：

\\[
\frac{\text{adc_value}}{\text{ADC_MAX}} = \frac{R_2}{R_1 + R_2}
\\]

---

## 步骤 3：求解 \( R_2 \)

为了求解 R2，我们将项 (R1 + R2) 移到等式的一边并重写。

\\[
R_2 = (R_1 + R_2) \times  \frac{\text{adc_value}}{\text{ADC_MAX}}
\\]

展开右侧：

\\[
R_2 =  R_1 \times \frac{\text{adc_value}}{\text{ADC_MAX}} + R_2 \times \frac{\text{adc_value}}{\text{ADC_MAX}}
\\]

将 R2 移到等式的一边以进行进一步操作：
\\[
R_2 - (R_2 \times \frac{\text{adc_value} }{\text{ADC_MAX}} )= R_1  \times \frac{\text{adc_value} }{\text{ADC_MAX}}
\\]

重新排列等式以分离 R2：

\\[
R_2 \left(1 - \frac{\text{adc_value}}{\text{ADC_MAX}}\right) = R_1 \times \frac{\text{adc_value} }{\text{ADC_MAX}}
\\]

展开表达式并消去分母中的 ADC_MAX：

\\[
\require{cancel}
R_2 \times \frac{\text{ADC_MAX} - \text{adc_value}}{{\text{ADC_MAX}}} = R_1 \times \frac{\text{adc_value} }{{\text{ADC_MAX}}}
\\]
<br/>
\\[
\require{cancel}
R_2 \times \frac{\text{ADC_MAX} - \text{adc_value}}{\cancel{\text{ADC_MAX}}} = R_1 \times \frac{\text{adc_value} }{\cancel{\text{ADC_MAX}}}
\\]

所以，我们得到：

\\[
R_2 \times (\text{ADC_MAX} - \text{adc_value}) = R_1 \times \text{adc_value}
\\]

最后将 (ADC_MAX−adc_value) 移到另一边得到 R2：

\\[
R_2 = R_1 \times \frac{\text{adc_value}}{\text{ADC_MAX} - \text{adc_value}}
\\]

---

## 最终公式

推导出的 R2 公式为：

\\[
R_2 = R_1 \times \frac{\text{adc_value}}{\text{ADC_MAX} - \text{adc_value}}
\\]

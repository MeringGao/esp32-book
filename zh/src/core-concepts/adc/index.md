# ADC（模数转换器，Analog to Digital Converter）

模数转换器（Analog-to-Digital Converter, ADC）是一种用于将模拟信号（如声音、光线或温度等连续信号）转换为数字信号（离散值，通常用 1 和 0 表示）的设备。这种转换对于微控制器（如 Raspberry Pi、Arduino）等数字系统与现实世界交互是必要的。例如，测量温度或声音的传感器产生模拟信号，需要将其转换为数字格式，以便数字设备进行处理。

<img style="display: block; margin: auto;" alt="ADC" src="../images/adc.jpg"/>

## ADC 分辨率（Resolution）
ADC 的分辨率（resolution）指的是 ADC 测量模拟信号的精细程度。它以位（bit）表示，分辨率越高，测量越精确。

- 8 位 ADC 产生的数字值范围是 0 到 255。
- 10 位 ADC 产生的数字值范围是 0 到 1023。
- 12 位 ADC 产生的数字值范围是 0 到 4095。

ADC 的分辨率可以用以下公式表示：
\\[
\\text{分辨率} = \\frac{\\text{Vref}}{2^{\\text{bits}} - 1}
\\]

## ESP32
ESP32 拥有 12 位模数转换器（ADC）。因此，它提供的值范围是 0 到 4095（共 4096 个可能值）。

\\[
\\text{分辨率} = \\frac{3.3V}{2^{12} - 1} = \\frac{3.3V}{4095} \\approx 0.000805 \\text{V} \\approx 0.8 \\text{mV}
\\]

## 引脚
//TODO: ESP32 ADC 引脚详情

## 分压电路中 ADC 值与 LDR 电阻的关系
在由 LDR（光敏电阻）和固定电阻组成的分压器（voltage divider）中，输出电压 \\( V_{\\text{out}} \\) 的计算公式为：

\\[
V_{\\text{out}} = V_{\\text{in}} \\times \\frac{R_{\\text{LDR}}}{R_{\\text{LDR}} + R_{\\text{fixed}}}
\\]

这与上一章解释的公式相同，只是将 \\({R_2}\\) 替换为 \\({R_{\\text{LDR}}}\\)，将 \\({R_1}\\) 替换为 \\({R_{\\text{fixed}}}\\)。

- **强光**（LDR 电阻低）：\\( V_{\\text{out}} \\) 降低，导致 ADC 值较低。
- **弱光**（LDR 电阻高）：\\( V_{\\text{out}} \\) 升高，导致 ADC 值较高。

## ADC 值计算示例：

**强光**：

 假设在强光下 LDR 的电阻值为 \\(1k\\Omega\\)（我们有一个 \\(10k\\Omega\\) 的固定电阻）。

\\[
V_{\\text{out}} = 3.3V \\times \\frac{1k\\Omega}{1k\\Omega + 10k\\Omega} \\approx 0.3V
\\]

ADC 值的计算为：
\\[
\\text{ADC 值} = \\left( \\frac{V_{\\text{out}}}{V_{\\text{ref}}} \\right) \\times (2^{12} - 1) \\approx \\left( \\frac{0.3}{3.3} \\right) \\times 4095 \\approx 372
\\]

**黑暗**：

  假设在非常弱的光线下 LDR 的电阻值为 \\(140k\\Omega \\)。

\\[
V_{\\text{out}} = 3.3V \\times \\frac{140k\\Omega}{140k\\Omega + 10k\\Omega} \\approx 3.08V
\\]

ADC 值的计算为：
\\[
\\text{ADC 值} = \\left( \\frac{V_{\\text{out}}}{V_{\\text{ref}}} \\right) \\times (2^{12} - 1) \\approx \\left( \\frac{3.08}{3.3} \\right) \\times 4095 = 3822
\\]

### **将 ADC 值转换回电压**：

现在，如果我们想将 ADC 值转换回输入电压，可以将 ADC 值乘以分辨率（0.8mV）。

例如，取 ADC 值为 3822：

\\[
\\text{电压} = 3822 \\times 0.8mV = 3057.6mV \\approx 3.06V
\\]


## 参考
- [什么是模数转换器及其工作原理](https://www.elprocus.com/analog-to-digital-converter/)

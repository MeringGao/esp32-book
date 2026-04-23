# LED PWM 控制器（LEDC）

ESP32 拥有 LED PWM 控制器（LEDC），用于生成 PWM 信号以控制 LED（例如调光效果）。然而，它的功能不仅限于 LED；你也可以将其用于其他应用。如果你不熟悉 PWM（脉冲宽度调制），我建议你先阅读[这里的 PWM 介绍](./index.md)。

LEDC 包含 16 个独立的 PWM 生成器，支持最大 20 位的 PWM 占空比分辨率（duty resolution）。这 16 个 PWM 通道进一步分为两类：8 个高速通道（high-speed channel）和 8 个低速通道（low-speed channel）。

> [!Note]
> **高速通道 vs 低速通道**：高速通道使用硬件自动以无 glitch 的方式调整 PWM 占空比，确保平滑运行。相比之下，低速通道依赖软件手动调整占空比。

PWM 控制器可以自动逐渐增加或减少占空比，实现平滑的渐变（fade）效果，而无需占用处理器资源。

## 时钟源（Clock Source）
微控制器中的时钟源就像是系统的"心跳"。它为微控制器提供规律的"滴答"，帮助它跟踪时间并协调所有任务。在 ESP32 中，你可以使用不同的时钟源来管理定时器。

<img style="display: block; margin: auto;" alt="PWM" src="../images/led-pwm-channels.png"/>
<span style="text-align: center;display: block; margin: auto;  ">图片来自 ESP32 技术参考手册</span>

有四个高速时钟模块可用，可以分配给高速通道。ESP32 中的高速定时器模块可以由 REF_TICK 或 APB_CLK 等时钟源驱动。在 esp-hal Rust 库中，这些定时器由 `timer::Number` 枚举表示，包括 `Timer0`、`Timer1`、`Timer2` 和 `Timer3`。

还有四个低速时钟模块可用，可以分配给低速通道。这些低速定时器可以由 REF_TICK 或 SLOW_CLOCK 驱动。SLOW_CLOCK 源可以是 APB_CLK（80 MHz）或 8 MHz 内部振荡器，通过 LEDC_APB_CLK_SEL 设置来管理这两个源之间的选择。

esp-hal 还定义了两个枚举：一个用于高速时钟源（`HSClockSource`），另一个用于低速时钟源（`LSClockSource`）。目前，两个枚举都只有一个条目 `APBClk`。

更多详情请参考 [ESP32 技术参考手册](https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf#ledpwm) 中的 LED PWM 控制器（LEDC）章节。

## 计算 PWM 占空比分辨率

在 ESP-HAL 中，配置定时器对象时需要同时指定占空比分辨率和频率。因此，了解如何从目标频率计算占空比分辨率，以及如何根据给定的占空比分辨率确定频率是很重要的。

以下公式来自 ESP32 技术参考手册，但变量名已简化。

### PWM 信号频率公式

PWM 信号的频率 \\( f_{\\text{pwm}} \\) 可以使用以下公式计算：

\\[
f_{\\text{pwm}} = \\frac{f_{\\text{LEDC\\_CLK}}}{\\text{clock_divider} \\cdot 2^{\\text{res_bits}}}
\\]

其中：
- \\( f_{\\text{LEDC\\_CLK}} \\) 是 PWM 定时器时钟源的频率（例如 APB_CLK、RC_FAST_CLK、REF_TICK）。
- \\( \\text{clock_divider} \\) 是时钟源的分频系数。
- \\( \\text{res_bits} \\) 是占空比分辨率，单位为位。


### PWM 占空比分辨率公式
这是从前一个公式推导出来的，用于计算所需的占空比分辨率。
\\[
\\text{res_bits} = \\log_2 \\left( \\frac{f_{\\text{LEDC\\_CLK}}}{f_{\\text{pwm}} \\cdot \\text{clock_divider}} \\right)
\\]

该公式给出的占空比分辨率以位为单位，表示 PWM 信号占空比可用的离散等级数量。

### 计算最高分辨率

当时钟分频器（\\( \\text{clock_divider} \\)）设置为 1 时，即不对时钟进行分频，达到**最高分辨率**。计算公式为：

\\[
\\text{最高分辨率} = \\log_2 \\left( \\frac{f_{\\text{LEDC\\_CLK}}}{f_{\\text{pwm}} \\cdot 1} \\right)
\\]

该计算给出了在给定时钟频率和 PWM 信号频率下，占空比可以使用的最大位数。

#### 示例：

对于 1 kHz 的 PWM 信号，APB_CLK 为 80 MHz：

\\[
\\text{最高分辨率} = \\log_2 \\left( \\frac{80,000,000}{1,000 \\cdot 1} \\right) = \\log_2(80,000) \\approx 16
\\]

因此，使用 80 MHz 时钟时，1 kHz PWM 信号的最高分辨率为 **16 位**。

### 计算最低分辨率
根据数据手册，分频系数范围为 1 ~ 1023。**最低分辨率**在时钟分频器达到最大值时计算。在这种情况下，时钟分频器为 \\( 1023 + \\frac{255}{256} \\)。最低分辨率计算如下：

\\[
\\text{最低分辨率} = \\log_2 \\left( \\frac{f_{\\text{LEDC\\_CLK}}}{f_{\\text{pwm}} \\cdot \\left( 1023 + \\frac{255}{256} \\right)} \\right)
\\]

该计算给出了在指定 PWM 信号频率下，占空比控制所需的最少位数。

#### 示例：

对于相同的 1 kHz PWM 信号，APB_CLK 为 80 MHz：

\\[
\\text{最低分辨率} = \\log_2 \\left( \\frac{80,000,000}{1,000 \\cdot 1023.996} \\right) = \\log_2 \\left( \\frac{80,000,000}{1,023,996} \\right) \\approx 6.28
\\]

因此，使用 80 MHz 时钟时，1 kHz PWM 信号的最低分辨率为 **7 位**。

### 常用 PWM 频率与分辨率

此表来自数据手册，总结了不同时钟源下常见 PWM 频率的最高和最低分辨率：

| **时钟源（Clock Source）** | **PWM 频率** | **最高分辨率（位）** | **最低分辨率（位）** |
|------------------|-------------------|-------------------------------|------------------------------|
| **APB_CLK (80 MHz)** | 1 kHz            | 16                            | 7                            |
| **APB_CLK (80 MHz)** | 5 kHz            | 13                            | 4                            |
| **APB_CLK (80 MHz)** | 10 kHz           | 12                            | 3                            |
| **RC_FAST_CLK (8 MHz)** | 1 kHz          | 12                            | 3                            |
| **RC_FAST_CLK (8 MHz)** | 2 kHz          | 11                            | 2                            |
| **REF_TICK (1 MHz)** | 1 kHz            | 9                             | 1                            |

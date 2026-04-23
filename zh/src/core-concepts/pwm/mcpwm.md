# 电机控制脉冲宽度调制器（Motor Control Pulse Width Modulator, MCPWM）

ESP32 有两个 PWM 外设：[LED 控制器（LEDC）](./led-pwm-controller.md)和 MCPWM。在本章中，我们将介绍 MCPWM。

MCPWM 专为电机和电源控制而设计。ESP32 具有两个 MCPWM 单元：MCPWM0 和 MCPWM1。下图提供了单个 MCPWM 模块的概览。

<img style="display: block; margin: auto;" alt="PWM" src="../images/esp32-mcpwm-module-overview.png"/>

每个 MCPWM 都有预分频器（prescaler，即时钟分频器）、三个定时器、三个运算器（operator）、故障处理程序（fault handler）和一个捕获模块（capture module）。

## 预分频器（Prescaler）
预分频器用于在将基础时钟频率应用到 PWM 信号之前降低它。MCPWM 由频率为 160 MHz 的时钟驱动，意味着它每秒滴答 1.6 亿次。这个时钟作为 MCPWM 模块的基础频率。然而，预分频器通过除法修改这个基础频率，有效地降低了 PWM 信号使用的时钟频率。

要计算每个时钟周期的持续时间，我们取频率的倒数：

\\[
\\text{周期} = \\frac{1}{160 \\times 10^6}  \\text{ s} = 0.00000000625  \\text{ s} = 6.25 \\text{ ns}
\\]

这意味着每个时钟周期持续 6.25 纳秒（ns）。

ESP32 中的 PWM_CLK_PRESCALE 寄存器（8 位）允许你通过除法调整这个基础时钟。一旦预分频器应用到基础时钟，它就会有效地降低时钟频率。产生的 PWM 周期使用以下公式计算：

\\[
\\text{PWM 时钟周期} = 6.25 \\text{ns} \\times (\\text{预分频器值} + 1)
\\]

**示例：**

假设你将预分频器值设置为 159。

\\[
\\text{PWM 时钟周期} = 6.25 \\text{ ns} \\times (159 + 1) = 6.25   \\text{ ns} \\times 160 = 1000 \\text{ ns}
\\]

要计算 PWM 频率，我们取周期的倒数：

\\[
\\text{PWM 频率} = \\frac{1}{\\text{PWM 时钟周期}} = \\frac{1}{1000 \\text{ ns}} = 1  \\text{ MHz}
\\]

因此，当预分频器值为 159 时，PWM 信号的周期为 1000 ns，频率为 1 MHz。

预分频器的最大值为 255（因为它是 8 位值），你可以达到的 PWM 频率为：

\\[
\\text{PWM 频率} = \\frac{1}{6.25 \\text{ ns} \\times (255 + 1)} = \\frac{1}{1600 \\text{ ns}} = 625 \\text{ KHz}
\\]


### 在 esp-hal 中

esp-hal 中的 `PeripheralClockConfig` 结构体负责 MCPWM 的时钟配置。它提供了两个函数，用于通过预分频器或频率进行初始化。

```rust
// 该函数自动计算预分频器以达到 1MHz
// This functions automatically calculate the prescaler to achieve the 1MHz
let clock_cfg = PeripheralClockConfig::with_frequency(1.MHz()).unwrap();
```

## 定时器（Timer）
定时器负责计数到指定值（称为"周期"），达到后复位并重新开始计数。这有助于控制输出信号的时序或频率。每个定时器都有一个 8 位时钟预分频器。

它具有 16 位计数器，可以在三种模式下运行：在 esp-hal 中，这表示为 `PwmWorkingMode` 枚举。
- **PwmWorkingMode::Increase**：向上计数，定时器从零开始向上计数到周期值，然后复位。
- **PwmWorkingMode::Decrease**：向下计数，定时器从周期值开始向下计数到零，然后复位。
- **PwmWorkingMode::UpDown**：向上-向下计数，定时器在向上计数和向下计数之间交替，创建一个对称的周期。

### 在 esp-hal 中

esp-hal 有一个 `TimerClockConfig` 结构体，你可以通过时钟配置实例来初始化它。你需要调用 `timer_clock_with_frequency` 函数（或 `timer_clock_with_prescaler`），参数如下：
1. 周期内的滴答数
2. PWM 工作模式
3. 目标频率

**示例：**

要以基础时钟频率 1 MHz（假设我们用 1 MHz 初始化了 `clock_cfg`）达到 50 Hz 频率（20 毫秒或 0.02 秒），我们需要计算 0.02 秒（20 毫秒）周期在 1 MHz 下的周期计数。计算如下：


\\[
\\text{计数} = 1,000,000 \\text{ Hz} \\times 0.02 \\text{ s} = 20,000
\\]

然后，我们可以调用函数来初始化 PWM 定时器：

```rust
let timer_clock_cfg = clock_cfg
    .timer_clock_with_frequency(19_999, PwmWorkingMode::Increase, 50.Hz())
    .unwrap();
```

我们传递 19,999 而不是 20,000，因为该函数内部会将其加 1，结果为 20,000。

## 运算器（Operator）
PWM 运算器使用 PWM 定时器的时序参考生成所需的输出波形。每个 PWM 运算器有两个输出：PWMxA 和 PWMxB，可以为上升沿和下降沿配置死区时间（dead-time）。

我们已配置 MCPWM0 外设，并选择 operator0 使用 PWMxA 输出。
```rust
let mut mcpwm = McPwm::new(peripherals.MCPWM0, clock_cfg);
let mut pwm_pin = mcpwm
	.operator0
	.with_pin_a(peripherals.GPIO33, PwmPinConfig::UP_ACTIVE_HIGH);
```

这里，`UP_ACTIVE_HIGH` 设置了 PWM 动作为 `UP_ACTIVE_HIGH`，PWM 更新方法为 `SYNC_ON_ZERO`。那么，这意味着什么？当定时器计数值小于时间戳值时，输出信号将被设置为高电平。然后，新的时间戳将在定时器达到零时应用。

### esp-hal 中的 set_timestamp 函数

`set_timestamp` 函数设置 PWM 信号应该改变的时间点。当定时器达到传递给该函数的值时，它会根据配置的 `PwmUpdateMethod` 更新 PWM 信号。在我们的示例中，我们在运算器中配置了 `UP_ACTIVE_HIGH`，因此输出将保持高电平，直到定时器计数值达到时间戳值。一旦达到时间戳值，信号将在 PWM 周期的剩余时间内变为低电平。

例如，如果我们将时间戳值设置为 500，信号将保持高电平，直到定时器计数达到 500。一旦达到 500，信号将在剩余的 19,500 计数（20,000 - 500）内保持低电平。

```rust
pwm_pin.set_timestamp(500);
```

## LEDC vs MCPWM

LEDC（LED 控制器）设计用于控制 LED，但也可以用于一般的 PWM 任务，使其成为 LED 调光或简单电机控制等基本应用的理想选择。

MCPWM（电机控制 PWM）专为电机控制而构建，具有死区时间插入和故障处理等高级功能，使其更适合精确和复杂的电机控制。

根据 API 参考文档，MCPWM 外设的典型用例包括：
- 数字电机控制，例如有刷/无刷直流电机、航模舵机（RC servo motor）
- 基于开关模式的数字电源转换
- 电源数模转换器（Power DAC），其中占空比等效于 DAC 模拟值
- 计算外部脉冲宽度，并将其转换为其他模拟值，如速度、距离
- 生成空间矢量脉宽调制（Space Vector PWM, SVPWM）信号，用于磁场定向控制（Field Oriented Control, FOC）

对于控制像 SG90 这样的 hobby 舵机，LEDC 简单且足够，但也可以使用 MCPWM。

## 参考

- [MCPWM API 参考](https://docs.espressif.com/projects/esp-idf/en/v4.4/esp32/api-reference/peripherals/mcpwm.html)，最新 [API 参考](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/peripherals/mcpwm.html)
- 更多详情请参考 [ESP32 技术参考手册](https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf#mcpwm) 第 417 页
- [ESP32 MCPWM 作为 SPWM 生成器](https://yopiediy.xyz/esp32-mcpwm-as-spwm-generator/)

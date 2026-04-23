## 使用 ESP32 的 LEDC 外设控制舵机

ESP32 有 LEDC 和电机控制脉宽调制器（MCPWM, Motor Control Pulse Width Modulator）外设用于 PWM 控制。首先，我们将使用 LEDC 来生成 PWM，我们在 LED、蜂鸣器和其他练习中已经使用过它了。

在之前的练习中，我们一直在使用 `set_duty` 函数，它接收占空比百分比作为 u8。但对于舵机，我们需要小数百分比（如 2.5%、7.5%、12.5%），这无法用 u8 表示。

### SetDutyCycle Trait

那么，我们该怎么办？幸运的是，esp-hal 中的 LEDC 通道实现了 embedded-hal 的 SetDutyCycle trait，让我们对 PWM 有更精细的控制。

我们将在 SetDutyCycle trait 中使用两个函数：`max_duty_cycle` 和 `set_duty_cycle`。
<br/>

**`max_duty_cycle`**：

此函数将根据我们在定时器中配置的占空比分辨率位（Duty Resolution）返回最大占空比。例如，如果我们将分辨率设置为 8 位，最大占空比将是 256（即 2<sup>8</sup>）。如果设置为 12 位，最大占空比将是 4096（即 2<sup>12</sup>）。我们将使用 12 位分辨率，因此最大占空比为 4096。

```rust
// 我们将其转换为 u32（从 u16），因为接下来的乘法需要 u32。
// We are converting to u32 (from u16) because we need u32 for the upcoming multiplication.
let max_duty_cycle = channel0.max_duty_cycle() as u32;
```
<br/>

**`set_duty_cycle`**：

此函数用于在 0 到 4096 的范围内设置占空比（针对 12 位分辨率）。

```rust
let duty = 512;
channel0.set_duty_cycle(duty).unwrap();
```

### 但是，怎么做？

这些函数接收或返回 u16 值。但我们如何使用百分比（小数）呢？我们不是直接使用百分比，而是计算对应的数值。例如，4096 的 2.5% 约为 102。这个值足以让舵机移动到正确的位置；在这种情况下，它会移动到 0 度。

为了从百分比计算数值，我们不会使用浮点数。相反，我们将最大占空比值转换为 `u32`。

像 2.5% 这样的百分比不能直接表示为 `u32`，因此我们将百分比乘以 10 使其适配。例如，2.5 变成 25。然后，我们将最终结果除以 1000（100 x 10）而不是 100。

\\[
\\text{min\\_duty} = \\frac{Percent_{u32} \\times \\text{max\\_duty\\_cycle}}{1000}
\\]

例如：
```rust
// 舵机位置的最小占空比（2.5%）
// Minimum duty (2.5%) for servo position
// 对于 12 位 -> 25 * 4096 /1000 => ~ 102
// For 12bit -> 25 * 4096 /1000 => ~ 102
// 它与 2.5 *4096 / 100 => ~102 相同
// it same as 2.5 *4096 / 100 => ~102
let min_duty = (25 * max_duty_cycle) / 1000;

// 舵机位置的最大占空比（12.5%）
// Maximum duty (12.5%) for servo position
// 对于 12 位 -> 125 * 4096 /1000 => 512
// For 12bit -> 125 * 4096 /1000 => 512
let max_duty = (125 * max_duty_cycle) / 1000;

```

### 从角度计算占空比

我们有一个简单的辅助函数，将角度转换为占空比值。我们需要传入角度（degree）、最小占空比（min_duty）以及舵机位置范围的最小和最大占空比之差。然后将最终值转换为 u16，因为 set_duty_cycle 函数只接受 u16。

```rust
let duty_gap = max_duty - min_duty; // 512 - 102 => 410
fn duty_from_angle(deg: u32, min_duty: u32, duty_gap: u32) -> u16 {
    let duty = min_duty + ((deg * duty_gap) / 180);
    duty as u16
}
```

例如，如果角度是 180 度。我们已经计算出最小和最大占空比范围是 102 和 512，它们之间的差是 410。让我们代入上面的公式。

duty = 102 + ((180 * 410) / 180) = 512

对于 90 度角度：

duty = 102 + ((90 * 410) / 180) = 307

307 约等于 4096 的 7.5%，这正是 90 度位置所需的值。

### 旋转

让我们将舵机的舵盘从 0 度旋转到 180 度，然后再回到 0 度，循环往复。我们首先根据角度计算占空比并设置占空比。我们等待 1500 毫秒，让舵机到达其位置。尝试将延时缩短到 50 毫秒，你会发现舵机开始产生抖动，并且根本无法到达预期位置。

```rust
loop{
    let duty = duty_from_angle(0, min_duty, duty_gap);
    channel0.set_duty_cycle(duty).unwrap();

    delay.delay_millis(1500); // 等待到达位置
                              // allow to reach its position

    let duty = duty_from_angle(180, min_duty, duty_gap);
    channel0.set_duty_cycle(duty).unwrap();

    delay.delay_millis(1500); // 等待到达位置
                              // allow to reach its position
}
```

不用担心如何运行这段代码。在下一章中，我们将看到一个 smoothly 将舵机从 0 度移动到 180 度，然后再回到 0 度的代码。

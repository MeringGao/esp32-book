# 延时（Delay）


仅仅在 LED 的高电平（High）和低电平（Low）状态之间切换是不够的。如果你这样做而不加任何延时，LED 会闪烁得非常快，你甚至都不会注意到。为了产生可见的闪烁效果，我们需要在切换之间添加一个短暂的停顿。

添加必要的导入：
```rust
use esp_hal::time::{Duration, Instant};
```

你可以这样做：

```rust

loop {
    led.toggle();
    blocking_delay(Duration::from_millis(500));
}
```

我们像这样定义延时函数：

```rust

fn blocking_delay(duration: Duration) {
    let delay_start = Instant::now();
    while delay_start.elapsed() < duration {}
}
```

这个 `blocking_delay` 函数只是在一个循环中等待，直到指定的持续时间过去。它使用 `Instant::now()` 记录开始时间，然后持续检查已经过了多少时间。这被称为忙等待（busy wait）或阻塞延时（blocking delay），因为 CPU 被困在循环中，在这段时间内不能做其他任何事情。虽然这不是最高效的方式，但它简单，并且在像闪烁 LED 这样的快速测试中效果很好。

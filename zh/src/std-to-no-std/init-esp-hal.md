# 初始化 ESP HAL

我们已经成功为 ESP32 准备好了一个二进制文件。然而，这个程序在这个阶段什么都不做。让我们通过复现快速入门部分中看到的那个闪烁程序来赋予它生命。

首先添加这个导入：

```rust
use esp_hal::clock::CpuClock;
```

现在，在你的 main 函数开头添加这一行：

```rust
let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
```

这一行为 HAL 创建了一个默认配置，并将 CPU 时钟设置为最大频率。你可以把它看作是给芯片一些关于它应该运行多快的指令。我们告诉它以最高速度运行。

## 初始化外设

接下来，我们使用 HAL 初始化外设：
```rust
let peripherals = esp_hal::init(config);
```

这个函数通过配置 CPU 时钟和看门狗定时器（watchdog timer）等项来设置系统。之后，它让我们可以访问芯片的外设和时钟，这样我们就可以在代码中开始使用它们了。

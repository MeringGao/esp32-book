## 异步（Async）

异步（async）编程允许任务并发运行而不会相互阻塞。在嵌入式系统（embedded systems）中，它使微控制器（microcontroller）能够处理多个任务，例如读取传感器或控制其他外设，而无需等待每个任务完成。你可以阅读 "[Rust 中的异步编程](https://rust-lang.github.io/async-book/intro.html)" 了解更多详情。

## Embassy

到目前为止，我们一直在阻塞模式（blocking mode）下运行代码。这意味着每当我们要求程序执行诸如延时（delay）一段时间或等待按钮按下等操作时，CPU 就会停止并等待该任务完成后再继续。这很容易理解，对于小型程序也很有效，但当我们想要同时处理多个任务时就会变得受限，比如读取传感器和监听输入；所有这些都不能相互阻塞。

这就是 [Embassy](https://github.com/embassy-rs/embassy) 的用武之地。Embassy 是一个为嵌入式系统设计的异步运行时（async runtime）。它允许我们使用 Rust 的 async 和 await 特性编写非阻塞（non-blocking）代码。任务可以暂停并让其他任务运行，而不是等待并浪费 CPU 时间，从而更好地利用处理器，并实现更响应迅速、更省电的应用程序。它可以与 ESP32、Pico 和其他微控制器一起使用。

例如，使用 Embassy，我们可以同时闪烁 LED 和监听触摸或按钮输入，而无需手动编写复杂的中断（interrupt）代码。

## HAL

Embassy 为多个微控制器系列提供支持异步的硬件抽象层（Hardware Abstraction Layers，HAL），提供安全且符合 Rust 惯用法的 API，让你无需处理底层寄存器即可与硬件交互。

官方 HAL 包括 embassy-stm32（STM32）、embassy-nrf（nRF52/53/54/91）、embassy-rp（RP2040）和 embassy-mspm0（TI MSPM0）。Embassy 还与社区 HAL 如 **esp-hal（ESP32）**、ch32-hal（CH32V）、mpfs-hal（PolarFire）和 py32-hal（Puya PY32）一起工作，使得在许多平台上编写可移植的异步代码变得容易。


## 开箱即用

Embassy 附带许多内置功能，使嵌入式开发更容易。例如，它包括 embassy-time 用于处理定时器和延时，embassy-net 用于网络支持，embassy-usb 用于构建 USB 设备功能等等。


## ESP RTOS

esp-hal 生态系统包含一个名为 `esp-rtos` 的 crate，提供 esp-hal 与 Embassy 异步框架之间的集成。

> 用于 esp-hal 的 RTOS（实时操作系统）实现。这个 crate 提供了在 esp-hal 之上运行异步代码所需的运行时，并实现了 esp-radio 所需的必要功能（线程、队列等）。


## 有用资源

- [Embassy 手册](https://embassy.dev/book/#_introduction)：Embassy 手册面向所有想要使用 Embassy 并了解 Embassy 工作原理的人。
- [Embassy GitHub](https://github.com/embassy-rs/embassy)
- [esp-rtos 文档](https://docs.rs/crate/esp-rtos/latest)

# Panic 处理程序（Panic Handler）

在这个阶段，当你尝试构建项目时，你会得到这个错误：

```sh
error: `#[panic_handler]` function required, but not found
```

当 Rust 程序发生 panic 时，通常由标准库中内置的 panic 处理程序来处理。但在上一步中，我们添加了 `#![no_std]`，它告诉 Rust 不要使用标准库。所以现在，默认情况下没有可用的 panic 处理程序。

在 no_std 环境中，你需要定义自己的 panic 行为，因为当出现问题时，没有操作系统或运行时来接管。

我们可以通过添加自己的 panic 处理程序来解决这个问题。只需创建一个带有 `#[panic_handler]` 属性的函数。该函数必须接受一个对 PanicInfo 的引用，并且其返回类型必须是 `!`，这意味着该函数永远不会返回。

将此添加到你的 src/main.rs 中：

```rust
#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}
```

## Panic crate

有一些现成的 crate 可以为 no_std 项目提供 panic 处理程序函数。一个简单且常用的 crate 是 "panic_halt"，它在发生 panic 时只是停止执行。


```rust
use panic_halt as _;
```
这一行引入了该 crate 中的 panic 处理程序。现在，如果发生 panic，程序就会停止并进入一个无限循环。

事实上，[panic_halt crate 的代码](https://github.com/korken89/panic-halt/blob/master/src/lib.rs)实现了一个简单的 panic 处理程序，看起来像这样：
```rust
use core::panic::PanicInfo;
use core::sync::atomic::{self, Ordering};

#[inline(never)]
#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {
        atomic::compiler_fence(Ordering::SeqCst);
    }
}
```

你可以使用像这样的外部 crate，或者手动编写你自己的 panic 处理程序函数。这取决于你。esp-template 工具实际上是将 panic 处理程序直接写入 main.rs 文件中，而不是引入一个单独的 crate。


**资源：**
- [Rust 官方文档](https://doc.rust-lang.org/nomicon/panic-handler.html)
- [嵌入式 Rust 之书](https://docs.rust-embedded.org/book/start/panicking.html)

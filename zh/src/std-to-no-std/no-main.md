# no_main

当你在这个阶段尝试构建时，你会得到一个错误，说 main 函数需要标准库。什么？！（我克制住了插入憨豆先生表情包的冲动，因为不是每个人都会喜欢表情包。）那现在怎么办？程序到底从哪里开始？

在嵌入式系统中，我们不使用依赖于标准库的常规 "fn main"。相反，我们必须告诉 Rust，我们将提供自己的入口点。为此，我们使用 no_main 属性。

`#![no_main]` 属性用于指示程序不会使用标准入口点（fn main）。

在你的 src/main.rs 文件顶部，添加这一行：

```rs
#![no_main]
```

## 声明入口点

现在我们已经选择不使用默认的入口点，我们必须告诉 Rust 从哪个函数开始。esp-hal crate 为此提供了一个方便的属性：`#[main]`。

这个属性将你的函数标记为自定义入口点。你用 `#[main]` 标记的函数将在 RAM 初始化完成后由复位处理程序（reset handler）调用。

首先，更新你的 Cargo.toml 以包含 esp-hal crate。由于它支持多种芯片，我们需要使用特性标志（feature flag）来启用我们正在使用的那一款（在我们的例子中是 esp32）。

```toml
[dependencies]
esp-hal = { version = "1.0.0", features = ["esp32"] }
```

然后，在你的 main.rs 中，像这样设置入口点：

```rust
use esp_hal::main;

#[main]
fn main() {}

```

但是等一下，还有一步。

入口函数必须具有特殊的签名——它应该永远不会返回。这意味着返回类型必须是 `!`（never 类型）。这告诉 Rust 你的函数将永远运行下去，这在嵌入式系统中很常见。

所以我们像这样更新 main 函数：

```rust
#[main]
fn main() -> ! {
    loop {}
}
```

## 我们到了吗？

万岁！现在尝试构建项目——它应该能成功编译。

你可以使用 file 命令检查生成的二进制文件：

```sh
file target/xtensa-esp32-none-elf/debug/std_to_no_std
```

它会显示类似这样的内容：

```sh
target/xtensa-esp32-none-elf/debug/std_to_no_std: ELF 32-bit LSB executable, Tensilica Xtensa, version 1 (SYSV), statically linked, with debug_info, not stripped
```
正如你所见，该二进制文件是为 32 位 Xtensa 目标构建的。这意味着我们的 ESP32 基础设置已经工作了。

但是我们到了吗？还没有完全。我们已经完成了一半——我们现在有了一个为 ESP32 准备的有效二进制文件，但在真正硬件上运行之前，还有更多的事情要做。


**资源：**
- [Rust 官方文档](https://doc.rust-lang.org/reference/crates-and-source-files.html?highlight=no_main#the-no_main-attribute)
- [用 Rust 编写操作系统](https://os.phil-opp.com/freestanding-rust-binary/#overwriting-the-entry-point)


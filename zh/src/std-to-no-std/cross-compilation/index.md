# 交叉编译（Cross Compilation）

Rust 允许你构建适用于与当前所用操作系统不同的操作系统的二进制文件。这意味着你可以在一个操作系统上编写代码，然后为完全不同的另一个操作系统构建二进制文件。例如，你可以在 Linux 上开发，并构建能在 Windows 上运行的 .exe 文件。你甚至可以面向 ESP32 或 STM32 这类裸机微控制器。在本节中，我们将探讨这是如何工作的，以及如何处理目标三元组（target triple）等概念。

> **太长不看（TL;DR）**
>
> 为 ESP32 构建二进制文件时，我们必须使用 "xtensa-esp32-none-elf" 作为目标。
>
> `cargo build --target xtensa-esp32-none-elf`
>
> 我们也可以在 `.cargo/config.toml` 中配置目标，这样就不需要每次都手动输入了。

## 为主机系统构建

假设我们使用的是 Linux 机器。当你运行常规的构建命令时，Rust 会为你的当前主机平台编译代码，在本例中就是 Linux：

```sh
cargo build
```

你可以使用 file 命令来确认它生成的二进制文件类型：
```sh
file ./target/debug/std_to_no_std
```

这将输出类似下面的内容。它告诉你这是一个 64 位的 ELF 二进制文件，采用动态链接，并且是为 Linux 构建的。

```sh
./target/debug/std_to_no_std: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, Build...
```

## 交叉编译到 Windows

现在假设你想在不离开 Linux 机器的情况下构建一个 Windows 二进制文件。这时就需要交叉编译（cross compilation）登场了。


首先，你需要告诉 Rust 目标平台是什么。你只需要执行一次：
```sh
rustup target add x86_64-pc-windows-gnu
```

这会添加使用 GNU 工具链（MinGW）生成 64 位 Windows 二进制文件的支持。

现在重新构建你的项目，这次指定目标：

```sh
cargo build --target x86_64-pc-windows-gnu
```
就是这样。Rust 现在会创建一个 Windows .exe 二进制文件，即使你仍然身处 Linux。输出的二进制文件将位于 `target/x86_64-pc-windows-gnu/debug/std_to_no_std.exe`

你可以这样检查文件类型：
```sh
file target/x86_64-pc-windows-gnu/debug/std_to_no_std.exe
```

它会给你类似这样的输出，一个用于 Windows 的 64 位 PE32+ 文件格式。
```sh
target/x86_64-pc-windows-gnu/debug/std_to_no_std.exe: PE32+ executable (console) x86-64, for MS Windows
```

## 什么是目标三元组（Target Triple）？

那么这个 `x86_64-pc-windows-gnu` 字符串到底是什么？

这就是我们所说的目标三元组（target triple），它告诉编译器你究竟想要什么样的输出。它通常遵循以下格式：

```html
`<架构>-<厂商>-<操作系统>-<ABI>`
```

但这个模式并不总是一致。有时 ABI 部分会缺失。在其他情况下，甚至连厂商或者厂商和 ABI 都可能不存在。结构可能变得很混乱，而且有很多例外情况。如果你想深入了解所有的怪癖和边缘情况，请查看参考文献中链接的文章 "What the Hell Is a Target Triple?"。

让我们来分解这个目标三元组的含义：

- **架构（x86_64）**：这表示 64 位 x86，也就是大多数现代 PC 使用的 CPU 类型。它也被称为 AMD64 或 x64。

- **厂商（pc）**：这基本上是一个占位符。在大多数情况下它并不重要。如果是 macOS，厂商名称会是 "apple"。

- **操作系统（windows）**：这告诉 Rust 我们想要构建在 Windows 上运行的东西。

- **ABI（gnu）**：这部分告诉 Rust 使用 GNU 工具链来构建二进制文件。



## 参考

- [Platform Support](https://doc.rust-lang.org/beta/rustc/platform-support.html)
- [Cross-Compilation](https://rust-lang.github.io/rustup/cross-compilation.html)
- [What the Hell Is a Target Triple?](https://mcyoung.xyz/2025/04/14/target-triples/)

# no_std

Rust 有两个主要的基础 crate：std 和 core。

- std crate 是标准库（Standard Library）。它提供了堆分配、文件系统访问、线程和 println! 等功能。

- core crate 是一个最小的子集。它只包含最基础的 Rust 特性，比如基本类型（Option、Result 等）、trait 和一些其他操作。它不依赖于操作系统或运行时。


当你在这个阶段尝试构建项目时，你会得到一堆错误。它看起来是这样的：

```sh
error[E0463]: can't find crate for `std`
  |
  = note: the `xtensa-esp32-none-elf` target may not support the standard library
  = note: `std` is required by `std_to_no_std` because it does not declare `#![no_std]`
  = help: consider building the standard library from source with `cargo build -Zbuild-std`

error: cannot find macro `println` in this scope
 --> src/main.rs:2:5
  |
2 |     println!("Hello, world!");
  |     ^^^^^^^

error: `#[panic_handler]` function required, but not found

For more information about this error, try `rustc --explain E0463`.
```

这里有很多错误。让我们逐个修复。第一个错误说目标可能不支持标准库。这是真的。我们已经知道了。问题是，我们没有告诉 Rust 我们不想使用 std。这就是 no_std 属性的作用所在。


## #![no_std]

`#![no_std]` 属性禁用了标准库（std）的使用。这对于嵌入式系统开发来说是必要的，因为嵌入式环境通常缺乏标准库所假设可用的许多资源（如操作系统、文件系统或堆分配）。

在你的 src/main.rs 文件顶部，添加这一行：

```rs
#![no_std]
```

就是这样。现在 Rust 知道这个项目将只使用 core 库，而不是 std。

## Println

`println!` 宏来自 [std crate](https://doc.rust-lang.org/std/macro.println.html)。由于我们在项目中不使用 std，所以我们不能使用 println!。让我们把它从代码中移除。

现在代码应该是这样的

```rust
#![no_std]
fn main() {
    
}
```

通过这个修复，我们已经解决了两个错误，并缩短了错误列表。仍然还有一个问题，我们将在下一节中修复它。


**资源：**
- [Rust 官方文档](https://doc.rust-lang.org/reference/names/preludes.html#the-no_std-attribute)
- [嵌入式 Rust 之书](https://docs.rust-embedded.org/book/intro/no-std.html)
- [用 Rust 编写操作系统](https://os.phil-opp.com/freestanding-rust-binary/#the-no-std-attribute)

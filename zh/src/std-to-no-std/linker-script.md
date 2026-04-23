# 链接器脚本（Linker Script）

你再次尝试构建项目了吗？是的，它不再能编译了。那么，发生了什么？

问题是：链接器不知道如何将你的程序布局到内存中。像 ESP32 这样的嵌入式系统没有操作系统来处理内存，所以我们必须告诉链接器代码、数据和栈应该放在哪里。

为了解决这个问题，我们提供一个链接器脚本（linker script）。这是一个简单的文本文件，告诉链接器诸如：

- 代码放在内存的哪个位置

- 栈从哪里开始

- 每个内存区域有多大

我们将使用的链接器脚本叫做 linkall.x。这个脚本由 [esp-hal](https://github.com/esp-rs/esp-hal/tree/8c64b09ba77d6ecb52eb73e7cea43d37a90d0dab/esp-hal/ld/esp32) 提供。它包含适合 ESP32 的内存布局设置。

## Cargo 配置

我们可以用以下行更新 ".cargo/config.toml" 文件来解决这个问题。然而，我们不会使用这种方法。相反，我们将使用 build.rs 文件。

```toml
[target.xtensa-esp32-none-elf]
rustflags = ["-C", "link-arg=-Tlinkall.x"]
```

> [!Important]
> 不要在 .cargo/config.toml 和 build.rs 中同时添加链接器参数。这会导致冲突并产生构建错误。只使用一种方法。

## Build.rs 文件

esp-generate 工具会将这一行添加到 build.rs 文件中。它还同时做了一些其他事情，我们这里不涉及。为了匹配那个结构，我们也将使用 build.rs 文件来添加链接器参数。


现在在你的项目目录根目录下（不在 src/ 文件夹内）创建一个名为 build.rs 的新文件，并添加以下代码：

```rust
fn main() {
    // 确保 linkall.x 是最后一个链接器脚本（否则可能会与 flip-link 产生问题）
    // make sure linkall.x is the last linker script (otherwise might cause problems with flip-link)
    println!("cargo:rustc-link-arg=-Tlinkall.x");
}
```

有了这个，你现在应该能够编译你的项目了：

```sh
cargo build
```

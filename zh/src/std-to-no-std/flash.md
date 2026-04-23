# 烧录（Flash）

不，我们不是在谈论世界上跑得最快的人。我们说的是将我们的二进制文件写入微控制器并运行它。

为此，我们将使用一个名为 espflash 的方便工具。还有其他工具如 probe-rs，但我们现在先坚持使用 espflash。


你可以使用以下命令烧录并监控你的程序：

```rust
espflash flash  --monitor --chip esp32 ./target/xtensa-esp32-none-elf/debug/std_to_no_std
```

## Cargo run

每次都要输入这么长的命令可能会很烦人。所以让我们通过更新 ".cargo/config.toml" 文件来简化它。我们可以告诉 Cargo 在我们运行 cargo run 时自动使用 espflash。

将此部分添加到你的 .cargo/config.toml：

```toml
[target.xtensa-esp32-none-elf]
runner = "espflash flash --monitor --chip esp32"
```

现在，你只需输入：

```sh
cargo run --release
```
你的程序就会被烧录并在 ESP32 上运行。


呼……我们从标准库二进制文件开始，将其转换为适用于 ESP32 的 no_std 二进制文件。终于，我们现在可以看到 LED 闪烁了。

好消息——我们每次创建新项目时都不必经历这个设置过程。这正是像 esp-generate（或 cargo-generate）这样的工具在嵌入式编程中非常有用的原因。它们设置了所有棘手的部分，这样我们就可以直接开始编写代码了。

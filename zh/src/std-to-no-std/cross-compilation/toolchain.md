# 工具链（Toolchain）

一旦你在 config.toml 文件中添加了目标并尝试使用 cargo build 构建项目，它将不再能够编译。此时，你会看到类似这样的错误：

```sh
'esp32' is not a recognized processor for this target (ignoring processor)
'esp32' is not a recognized processor for this target (ignoring processor)
error[E0463]: can't find crate for `std`
  |
  = note: the `xtensa-esp32-none-elf` target may not be installed
  = help: consider downloading the target with `rustup target add xtensa-esp32-none-elf`

For more information about this error, try `rustc --explain E0463`.
error: could not compile `std_to_no_std` (bin "std_to_no_std") due to 1 previous error
```

消息说 esp32 不是可识别的处理器，并且还抱怨目标可能没有安装。但是等一下，我们不是已经用 espup 工具安装好一切了吗？是的，我们确实安装了。

那么发生了什么？默认情况下，Rust 使用的是你系统的全局工具链。那个工具链对 ESP32 一无所知。espup 安装了一个名为 "esp" 的独立工具链（你也可以给它起不同的名字）。我们只需要告诉 Rust 使用它。

在你的项目根目录下，创建一个名为 rust-toolchain.toml 的文件，并添加以下内容：

```toml
[toolchain]
channel = "esp"
```

你可以使用自定义工具链来代替默认的 "esp" 工具链。在本书中，我使用了一个名为 "book-1.0.0" 的自定义工具链。只要确保你的工具链已正确安装，并且你使用的是确切的名称；否则，你会看到一个错误提示说它未安装。

## Build-std

再次尝试构建。它仍然无法编译，但这次的错误看起来有些不同。现在它只是说找不到 std 的 crate。

```sh
error[E0463]: can't find crate for `std`
  |
  = note: the `xtensa-esp32-none-elf` target may not be installed
  = help: consider downloading the target with `rustup target add xtensa-esp32-none-elf`
  = help: consider building the standard library from source with `cargo build -Zbuild-std`

For more information about this error, try `rustc --explain E0463`.
error: could not compile `std_to_no_std` (bin "std_to_no_std") due to 1 previous error
```

问题是，ESP32 目标是一个 [Tier 3 目标](https://doc.rust-lang.org/beta/rustc/target-tier-policy.html#tier-3-target-policy)。这意味着 Rust 不会为它提供预编译的标准库。我们必须自己从源代码构建 core 库。

为此，请用以下内容更新 .cargo/config.toml 文件：
```toml
[unstable]
build-std = ["core"]
```

再试一次构建。它仍然无法编译。但别担心，我们离成功越来越近了。在下一节中，我们将修复下一组错误。

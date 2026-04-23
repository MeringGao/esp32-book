# 开发环境（Development Environment）

[官方文档](https://docs.espressif.com/projects/rust/book/getting-started/index.html) 提供了更全面的安装说明。不过，我将快速介绍我们的练习所需的必备工具和设置。如果你遇到任何问题，请参阅官方文档进行故障排除。

## cargo-binstall

这是为了在不从源码构建的情况下安装 Rust 二进制文件，无需使用 `cargo install` 或手动下载包，你可以使用 cargo-binstall。我们将使用这个工具来安装 espflash。

```rust
cargo install cargo-binstall
```

## espflash

"espflash 是一个基于 esptool.py 的串行烧录（Serial Flasher）工具，用于乐鑫的 SoC 和模块。" 这将是我们在不使用 probe-rs 时将代码放入设备并运行的工具。

```sh
cargo binstall espflash
```

如果你遇到任何问题，可以尝试安装本书编写时使用的确切版本。

```sh
cargo binstall espflash@4.2.0
```

安装完成后，输入 espflash 命令以验证其是否正常工作。

```sh
espflash --version
```

## ESP-RS 提供的模板

我们将使用 [ESP-RS](https://docs.espressif.com/projects/rust/book/getting-started/tooling/esp-generate.html) 提供的模板，它们提供两套模板：

- **esp-generate**：`no_std` 模板。这是我们大部分时间要关注的。
- **esp-idf-template**：`std` 模板。

## esp-generate

**esp-generate** 工具用于创建 `no_std` 应用。目前，它支持 ESP32、ESP32-C2/C3/C6、ESP32-H2 和 ESP32-S2/S3。

```sh
cargo install esp-generate --locked
```

如果你想完全按照本项目中的代码进行，请安装用于生成示例的这个 esp-generate 版本：
```sh
cargo install esp-generate@1.0.0 --locked
```


### 使用 `esp-generate` 创建项目

对于本书，我们将使用 ESP32。我强烈建议使用相同的硬件，以便更容易跟上。

```sh
# 将 PROJECT_NAME 和 --chip 替换为你特定的芯片和项目名称。
esp-generate --chip esp32 PROJECT_NAME
```


## RISC-V 和 Xtensa 目标的工具链（Toolchains）

你还需要 `espup` 来安装必要的工具链（Toolchains）。你可以在 [这里](https://docs.espressif.com/projects/rust/book/getting-started/toolchain.html?highlight=espup#xtensa-devices) 找到详细信息。

```sh
cargo binstall espup
```

或者本书编写时使用的确切版本：
```sh
cargo binstall espup@0.16.0
```

然后安装工具链（Toolchain）
```sh
espup install
```

如果你想按原样使用我创建的项目，你可能需要确切的 Rust 工具链（Toolchain）版本。你可以使用以下命令：
```sh
espup install --toolchain-version 1.90.0
```

**注意：** 仅当你计划克隆并完全按原样运行项目示例时，才安装这个特定版本。使用不匹配的版本可能会导致奇怪的错误（例如错误：asm! 宏不允许在 naked 函数中使用）。


## 不做修改地使用项目示例

当你使用 esp-generate 创建项目时，它会自动将 "esp" 设置为工具链通道。如果你想"克隆"并使用现有项目而不是从头创建一个，你需要将工具链名称指定为 "book-1.0.0"（因为项目的 rust-toolchain.toml 配置的工具链名称为 book-1.0.0）。这仅适用于你从 esp32-projects 仓库克隆项目并希望不做任何修改地运行它的情况。

```sh
espup install --name book-1.0.0 --toolchain-version 1.90.0
```

## Shell 环境中的 ESP 工具链（Toolchain）

如果你使用 Fish shell，将以下行添加到你的 Fish 配置中：

```sh
echo '. ~/export-esp.sh' >> ~/.config/fish/config.fish
```

如果你使用 bash，将以下行添加到你的 ~/.bashrc 文件中：

```sh
echo '. ~/export-esp.sh' >> ~/.bashrc
```

验证编译器路径。检查工具链（Toolchain）现在是否可见：
```sh
which xtensa-esp32-elf-gcc
```

## USB 访问权限

要烧录（Flash）你的 ESP32，你的计算机需要有权访问 USB 串口。运行此命令以授予访问权限：

```sh
sudo usermod -a -G dialout $USER
```

注销并重新登录以使更改生效。

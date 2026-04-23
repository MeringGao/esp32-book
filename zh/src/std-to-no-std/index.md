# 从 std 到 no_std

我们已经成功烧录并运行了第一个程序，实现了一个闪烁效果。不过，我们还没有详细研究代码或项目结构。在本节中，我们将从零开始重新创建相同的项目，而不是使用模板。我会逐步解释每一部分代码和配置。准备好迎接挑战了吗？


## 创建一个新项目

我们首先创建一个标准的 Rust 二进制项目。使用以下命令：

```rust
cargo new std_to_no_std
```

在这个阶段，项目中会包含预期的常规文件。

```sh
├── Cargo.toml
└── src
    └── main.rs
```

我们的目标是达到以下最终项目结构：

```sh
├── build.rs
├── .cargo
│   └── config.toml
├── Cargo.toml
├── rust-toolchain.toml
├── src
│   ├── bin
│   │   └── main.rs
```


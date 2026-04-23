# 帮助与故障排除（Help & Troubleshooting）

如果你在练习过程中遇到任何错误（Bug）、报错或其他问题，以下是一些排查和解决问题的方法。

## 1. 与可运行的代码进行比较

检查完整的代码示例或克隆参考项目进行比较。仔细审查你的代码和 `Cargo.toml` 依赖版本。留意任何语法或逻辑错误。如果某个必需的特性（Feature）未启用或存在特性不匹配，请确保按照练习中所示启用正确的特性。

如果你发现版本不匹配，要么调整你的代码（研究并找到解决方案；这是学习和理解事物的绝佳方式）以使其适用于新版本，要么更新依赖项以匹配教程中使用的版本。

## 2. 搜索或报告 GitHub Issues

访问 GitHub issues 页面，看看是否有人遇到过相同的问题：
[https://github.com/ImplFerris/esp32-book/issues?q=is%3Aissue](https://github.com/ImplFerris/esp32-book/issues?q=is%3Aissue)

如果没有，你可以提交一个新 issue 并清楚地描述你的问题。

## 3. 向社区求助

Rust 嵌入式社区在 Matrix Chat 中非常活跃。Matrix 聊天是一个开放的安全、去中心化通信网络。

以下是一些与本书涵盖主题相关的有用 Matrix 频道：

- **Embedded Devices Working Group**
  [`#rust-embedded:matrix.org`](https://matrix.to/#/#rust-embedded:matrix.org)
  关于使用 Rust 进行嵌入式开发的一般讨论。

- **ESP32 Development**
  [`#esp-rs:matrix.org`](https://matrix.to/#/#esp-rs:matrix.org)
  专注于 ESP32 系列芯片的 Rust 开发。

- **Debugging with Probe-rs**
  [`#probe-rs:matrix.org`](https://matrix.to/#/#probe-rs:matrix.org)
  用于支持和讨论 [probe-rs](https://probe.rs) 调试工具包。

- **Embedded Graphics**
  [`#rust-embedded-graphics:matrix.org`](https://matrix.to/#/#rust-embedded-graphics:matrix.org)
  用于使用 [`embedded-graphics`](https://docs.rs/embedded-graphics)，一个用于嵌入式系统的绘图库。

你可以创建一个 Matrix 账户并加入这些频道，以从有经验的开发者那里获得帮助。

你可以在 [Awesome Embedded Rust - Community Chat Rooms section](https://github.com/rust-embedded/awesome-embedded-rust?tab=readme-ov-file#community-chat-rooms) 中找到更多社区聊天室。


## 4. Discord

有一个非官方的嵌入式 Rust Discord 社区，你可以在那里提问、讨论话题、分享你的经验并展示你的项目。它尤其对学习者和一般讨论很有用。

请记住，大多数 HAL 和嵌入式生态系统的维护者在 Matrix 上更活跃。不过，这个 Discord 服务器仍然是一个学习和与他人互动的好地方。

在此加入：[https://discord.gg/NHenanPUuG](https://discord.gg/NHenanPUuG)

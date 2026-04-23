# 使用 Rust 和 ESP32 在无源蜂鸣器上播放歌曲

我们将在蜂鸣器上播放歌曲。如果你对音符和五线谱不太了解，可以查看我提供的[快速乐理介绍](./music-theory.md)。

我将代码拆分成了 Rust 模块（module）（你也可以像之前那样在单个文件中完成）：[`music`](./music-module.md)、[`got`](./got-module.md)。

本练习建议使用无源蜂鸣器（passive buzzer），不过你也可以使用有源蜂鸣器或无源蜂鸣器。

## PWM
我们将使用 PWM 来调整发送到蜂鸣器的信号频率，每个频率对应一个音符。频率（音符）将持续特定时长，然后根据乐谱切换到下一个音符。

例如，音符 "A4" 是 440Hz，持续 X 时长。我们在 PWM 中设置这个频率，并添加 X 时长的延时。

我建议阅读 [PWM 章节](../../core-concepts/pwm/index.md) 来熟悉 PWM 的工作原理。


## 歌曲仓库

在本练习中，我们将播放 Pink Panther 主题曲。不过，你可以参考我创建的 rust-embedded-songs [仓库](https://github.com/ImplFerris/rust-embedded-songs/)，也可以使用其他歌曲。


## lib.rs 子模块
更新 lib.rs 以定义子模块，然后创建 `music.rs` 和 `pink_panther.rs` 文件。

```rust
#![no_std]
pub mod music;
pub mod pink_panther;
```

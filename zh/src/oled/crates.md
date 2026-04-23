# 相关 Crate

我们将主要使用两个 crate 来控制 OLED 显示屏：[ssd1306](https://docs.rs/ssd1306/latest/ssd1306/) 和 [embedded_graphics](https://docs.rs/embedded-graphics/latest/embedded_graphics/)。

## SSD1306 OLED 显示屏驱动

这个 crate 为 SSD1306 单色 OLED 显示屏提供了驱动接口，通过 [display_interface](https://docs.rs/display-interface/latest/display_interface/) crate 支持 I2C 和 SPI。它还支持异步（async），但需要通过特性标志（feature flag）启用。

### 添加带异步支持的 ssd1306

```toml
ssd1306 = { version = "0.10.0", features = ["async"] }
```

### 添加不带异步支持的 ssd1306

```toml
ssd1306 = { version = "0.10.0", features = [] }
```

## Embedded Graphics

要在 OLED 显示屏上显示文本或绘制图像，我们将结合使用 embedded_graphics crate 和 ssd1306 crate。

"Embedded-graphics 是一个专注于内存受限嵌入式设备的 2D 图形库。"

"embedded-graphics 的核心目标是在不使用任何缓冲区的情况下绘制图形；该 crate 兼容 no_std，无需动态内存分配器（heap allocator），也无需预先分配大块内存。为了实现这一点，它采用基于迭代器（Iterator）的方法，实时计算像素的颜色和位置，并保存最少的状态。这使得使用它的应用程序可以以更少甚至零性能损失的代价使用更少的 RAM。"

你可以在使用不同类型的 OLED 模块时，将这个 crate 与各种 OLED 显示屏和驱动配合使用。[文档](https://docs.rs/embedded-graphics/latest/embedded_graphics/)详细介绍了其功能和支持的驱动，建议通读一遍。

```toml
embedded-graphics = "0.8.1"
```

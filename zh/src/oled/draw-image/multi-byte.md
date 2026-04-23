## 使用多个字节表示更宽的像素宽度

在之前的示例中，我们通过使用 8 像素宽的图像来保持简单，使得每一行都可以用单个字节表示。然而，在实际场景中，我们可能需要更多的像素。那么，如何表示它们呢？我们可以使用多个字节来表示更宽的像素宽度。但是，等等；如果这样做，系统如何区分列和行呢？

这正是我们需要在 embedded graphics crate 中指定确切宽度的原因。通过指定宽度，系统就知道需要用多少个数组元素来表示宽度。基于这个宽度大小（以及图像格式），系统就可以确定高度。

例如，让我们考虑一个分辨率为 31x7 像素的图像。宽度为 31 像素，每个像素由 1 位表示。要用字节来表示 31 个像素，我们需要计算需要多少个字节。由于一个字节是 8 位，我们将总像素数（31）除以 8。这给了我们 3 个完整的字节来表示 24 个像素，还需要额外一个字节来存储剩余的 7 个像素。因此，表示 31 个像素需要 4 个字节。

<img style="display: block; margin: auto;" title="ohm symbol 1bpp image format" src="../images/embedded-graphics-multi-byte-oled.png"/>

embedded graphics crate 在内部使用以下代码片段来计算高度。我们不会在自己的代码中包含这段代码，但我在这里展示它以供参考，说明其内部工作原理：

```rust
let height = data.len() / bytes_per_row(width, C::Raw::BITS_PER_PIXEL);
//...
//...
const fn bytes_per_row(width: u32, bits_per_pixel: usize) -> usize {
    (width as usize * bits_per_pixel + 7) / 8
}
```

这里，数据的长度为 28（数组元素个数），每像素位数为 1，图像宽度为 31。如果你应用这个逻辑，会得到 `bytes_per_row` 为 4，高度为 7。

你可以直接运行以下代码，或者在 Rust Playground 中运行，以理解其背后的逻辑：

```rust
// 31x7 像素
#[rustfmt::skip]
const IMG_DATA: &[u8] = &[
    // 第 1 行
    0b00000001,0b11111111,0b11111111,0b00000000,
    // 第 2 行
    0b00000001,0b11111111,0b11111111,0b00000000,
    // 第 3 行
    0b00000001,0b10000000,0b00000011,0b00000000,
    // 第 4 行
    0b11111111,0b10000000,0b00000011,0b11111110,
    // 第 5 行
    0b00000001,0b10000000,0b00000011,0b00000000,
    // 第 6 行
    0b00000001,0b11111111,0b11111111,0b00000000,
    // 第 7 行
    0b00000001,0b11111111,0b11111111,0b00000000,
];

const fn bytes_per_row(width: u32, bits_per_pixel: usize) -> usize {
    (width as usize * bits_per_pixel + 7) / 8
}

fn main(){
    const BITS_PER_PIXEL: usize = 1;
    let width = 31;
    let data = IMG_DATA;

    println!("Bytes Per Row:{}", bytes_per_row(width,BITS_PER_PIXEL));
    let height = data.len() / bytes_per_row(width, BITS_PER_PIXEL);
    println!("Height: {}", height);
}
```

你不需要手动创建这些字节数组，可以使用像 [imag2bytes](https://implferris.github.io/image2bytes/) 这样的在线工具来为你生成字节数组。

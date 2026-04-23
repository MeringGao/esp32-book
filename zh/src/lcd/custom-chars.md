## 自定义字符（Custom Characters）

除了支持的字符外，你还可以创建自己的自定义字符，例如笑脸或心形符号。该模块包含 64 字节的字符生成器 RAM（Character Generator RAM，CGRAM），最多允许 8 个自定义字符。

每个字符是一个 8x8 网格，其中每一行由一个 8 位值（`u8`）表示。这样每个字符占 8 字节（8 行 × 每行 1 字节）。这就是为什么总共 64 字节的情况下，你最多只能存储 8 个自定义字符（8 个字符 × 8 字节 = 64 字节）。

<img style="display: block; margin: auto;width:400px;" alt="custom characters grid" src="./images/custom-character-grid-bits.jpg"/>

注意：如果你还记得，在我们的 LCD 模块中，每个字符表示为 5x8 网格。但是等等，我们不是说字符需要 8x8 网格吗？是的，没错——我们需要 8 × 8（8 字节）的存储空间，但每一行我们只使用 5 位。每一行中的 3 个高位留为零。


## 支持的字符（Supported Characters）

查阅 HD44780 数据手册（datasheet）时，你会发现两个字符集表，对应两种不同的 ROM 版本（A00 和 A02）。要确定你的显示屏使用哪种 ROM，请尝试两个表中的独特字符。能正确显示的那个即表明 ROM 版本。一旦确定，你只需要参考相关的表格即可。

就我而言，我使用的 LCD 模块基于 ROM 版本 A00。我将展示 A00 表格并解释如何解读它，尽管解读逻辑对两个版本都相同。

<img style="display: block; margin: auto;" alt="lcd1602" src="./images/lcd1602-characters-set.png"/>

它是一个 8 位字符，高 4 位在前，低 4 位在后，组成完整的字符字节。在参考表中，高 4 位对应列，低 4 位对应行。

例如，要获取字符 "#" 的二进制表示，高 4 位是 0010，低 4 位是 0011。将它们组合起来得到完整的二进制值 00100011。在 Rust 中，你可以用二进制（0b00100011）或十六进制（0x23）表示这个值。

### hd44780-driver crate

在我们使用的 `hd44780-driver` crate 中，我们可以直接将字符作为单个字节或字节序列写入。

#### 写入单个字节
```rust
lcd.write_byte(0x23, &mut timer).unwrap();
lcd.write_byte(0b00100011, &mut timer).unwrap();
```

#### 写入多个字节
```rust
lcd.write_bytes(&[0x23, 0x24], &mut timer).unwrap();
```

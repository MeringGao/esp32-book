# MIFARE

MIFARE 是 NXP Semiconductors 开发的一系列用于非接触式智能卡（contactless smart card）和 proximity card 的集成电路（IC）芯片。MIFARE 卡片遵循 ISO/IEC 14443A 标准，并使用 Crypto-1 算法等加密方法。最常见的系列是 MIFARE Classic，其子类型为 [MIFARE Classic EV1](https://www.nxp.com/products/rfid-nfc/mifare-hf/mifare-classic/mifare-classic-ev1-1k-4k:MF1S50YYX_V1)。

## 存储布局

MIFARE Classic 1K 卡分为 16 个扇区（sector），每个扇区包含 4 个块（block）。每个块最多可存储 16 字节，总存储容量为 1KB。

16 扇区 × 4 块/扇区 × 16 字节/块 = 1024 字节 = 1KB

<a href="./images/mifare-memory-layout.png"><img style="display: block; margin: auto;" alt="MIFARE Memory layout" src="./images/mifare-memory-layout.png"/></a>

### 扇区尾部块（Sector Trailer）

每个扇区的最后一个块被称为"尾部块（trailer block）"，它保存了两个密钥和该扇区内各块的可编程访问条件。每个扇区都有自己的一对密钥（KeyA 和 KeyB），支持使用密钥层级结构的多个应用。

> [!Note]
> MIFARE Classic 1K 卡预配置的默认密钥为 FF FF FF FF FF FF，KeyA 和 KeyB 均如此。读取尾部块时，KeyA 的值会全部返回为零（00 00 00 00 00 00），而 KeyB 则按原值返回。

默认情况下，访问字节（尾部块的第 6、7、8 字节）设置为 FF 07 80h。你可以参考[数据手册](https://www.nxp.com/docs/en/data-sheet/MF1S50YYX_V1.pdf)的第 10 页获取更多信息。第 9 个字节可用于存储数据。

<table border="1" cellspacing="0" cellpadding="5" style="border-collapse: collapse; width: 100%; text-align: center;">
  <thead>
    <tr>
      <th rowspan="2" style="background-color: #607D8B; color: #000;">字节编号</th>
    </tr>
    <tr>
      <th style="background-color: #1B5E20;">0</th>
      <th style="background-color: #1B5E20;">1</th>
      <th style="background-color: #1B5E20;">2</th>
      <th style="background-color: #1B5E20;">3</th>
      <th style="background-color: #1B5E20;">4</th>
      <th style="background-color: #1B5E20;">5</th>
      <th style="background-color: #FF6F00;">6</th>
      <th style="background-color: #FF6F00;">7</th>
      <th style="background-color: #FF6F00;">8</th>
      <th style="background-color: #FF6F00;">9</th>
      <th style="background-color: #2962FF;">10</th>
      <th style="background-color: #2962FF;">11</th>
      <th style="background-color: #2962FF;">12</th>
      <th style="background-color: #2962FF;">13</th>
      <th style="background-color: #2962FF;">14</th>
      <th style="background-color: #2962FF;">15</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="background-color: #607D8B; color: #000;">描述</td>
      <td colspan="6" style="background-color: #1B5E20; color: #000;">KEY A</td>
      <td colspan="3" style="background-color: #FF6F00; color: #000;">访问位（Access Bits）</td>
      <td style="background-color: #FF6F00; color: #000;">用户数据</td>
      <td colspan="6" style="background-color: #2962FF; color: #000;">KEY B</td>
    </tr>
    <tr>
      <td style="background-color: #607D8B; color: #000;">默认数据</td>
      <td style="background-color: #1B5E20;">FF</td>
      <td style="background-color: #1B5E20;">FF</td>
      <td style="background-color: #1B5E20;">FF</td>
      <td style="background-color: #1B5E20;">FF</td>
      <td style="background-color: #1B5E20;">FF</td>
      <td style="background-color: #1B5E20;">FF</td>
      <td style="background-color: #FF6F00;">FF</td>
      <td style="background-color: #FF6F00;">07</td>
      <td style="background-color: #FF6F00;">80</td>
      <td style="background-color: #FF6F00;">69</td>
      <td style="background-color: #2962FF;">FF</td>
      <td style="background-color: #2962FF;">FF</td>
      <td style="background-color: #2962FF;">FF</td>
      <td style="background-color: #2962FF;">FF</td>
      <td style="background-color: #2962FF;">FF</td>
      <td style="background-color: #2962FF;">FF</td>
    </tr>
  </tbody>
</table>

### 制造商块（Manufacturer Block）

第一个扇区（sector 0）的第一个块（block 0）包含 IC 制造商的数据，包括 UID。该块为写保护。

### 数据块（Data Block）

每个扇区都有一个尾部块（trailer block），因此每个扇区只有 3 个块可用于数据存储。但是，第一个扇区只有 2 个可用块，因为第一个块存储了制造商数据。

要读取或写入数据，首先需要使用对应扇区的 Key A 或 Key B 进行认证（authenticate）。

数据块可以根据访问位（access bits，稍后我们会解释）进一步分为两类：
- 读写块（read/write block）：标准数据块，允许基本的读写操作。
- 数值块（value block）：这类块非常适合电子钱包等应用，通常用于存储数值，如账户余额。因此，你可以执行增值（例如，余额增加 10 元）或减值（例如，交易扣除 5 元）操作。

## 参考资料

- 数据手册：[MIFARE Classic EV1 1K - Mainstream contactless smart card IC for fast and easy solution development](https://www.nxp.com/docs/en/data-sheet/MF1S50YYX_V1.pdf)

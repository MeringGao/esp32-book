# RFID

在本节中，我们将使用 RFID 读卡器（RC522）模块来读取 RFID 标签（RFID tag）和钥匙扣标签（key fob）中的数据。

你可能在日常生活中已经接触过这些技术：用智能钥匙打开公寓门禁、用标签进入办公室、驶入智能停车场，或者使用非接触式信用卡（contactless credit card）付款。如果你用过酒店房卡或车载电子收费标签（toll pass），那么你已经体验过 RFID 的实际应用了！

<div style="text-align: center; margin: 20px 0;">
  <img style="display: block; margin: auto; " alt="MIFARE Memory layout" src="./images/rfid-card-tag.jpg"/>
  <div style="margin-top: 10px; font-style: italic; color: #666;">
    Photo credits to Security Instrument Corp
  </div>
</div>

RFID（Radio Frequency Identification，射频识别）是一种利用无线电波来识别和追踪物体、动物的技术。当标签（tag）进入读卡器范围时，它会无线传输存储在标签芯片和天线中的数据。

## 按距离分类

RFID 系统可以根据其工作频率进行分类。三种主要类型如下：

- **低频（LF, Low Frequency）**：工作频率约为 125 kHz，读取距离短（最长 10 cm）。速度较慢，常用于门禁控制和牲畜追踪。

- **高频（HF, High Frequency）**：工作频率为 13.56 MHz，读取距离为 10 cm 到 1 m。速度适中，广泛用于门禁系统（如办公空间、公寓、酒店房卡）、票务、支付和数据传输等场景。我们将要使用的 RC522 模块工作频率就是 13.56 MHz。

- **超高频（UHF, Ultra-High Frequency）**：工作频率为 860–960 MHz，读取距离可达 12 m。速度更快，常用于零售库存管理、防伪和物流领域。

## 按供电方式分类

RFID 标签根据其供电方式可分为主动式（active）和被动式（passive）。

- **主动式标签（Active tags）**：自带电池，可以主动发送信号。通常用于大型物体，如铁路车厢、大型可重复使用集装箱以及需要长距离追踪的资产。
- **被动式标签（Passive tags）**：与主动式标签不同，被动式标签没有电池。它们依靠 RFID 读卡器发出的电磁场来供电。一旦被激活，它们就会通过无线电波传输数据。这是最常见的 RFID 标签类型，也是你在日常生活中最有可能遇到的。如果你猜对了，没错，RC522 使用的就是被动式标签。

## 组成部分

RFID 系统由 RFID 读卡器（RFID reader）组成，技术上称为 PCD（Proximity Coupling Device，近耦合设备）。在被动式 RFID 标签系统中，读卡器通过电磁场为标签供电。标签本身被称为 RFID 标签（RFID tag），技术术语为 PICC（Proximity Integrated Circuit Card，近耦合集成电路卡）。了解这些技术术语也很有用，当你需要查阅数据手册（datasheet）和其他文档时会派上用场。

读卡器通常包含 FIFO 缓冲区（FIFO buffer）和 EEPROM 等存储组件。它们还集成了加密功能，以确保与标签之间的安全通信，只允许经过认证的 RFID 读卡器与之交互。例如，NXP Semiconductors 的 RFID 读卡器使用 Crypto-1 密码算法进行认证。

每个 RFID 标签都有一个硬编码的 UID（Unique Identifier，唯一标识符），大小可以是 4、7 或 10 字节。

## 参考资料

- [What Is A RFID Antenna?](https://www.sannytelecom.com/what-is-a-rfid-antenna/)
- [Types of RFID Systems](https://www.impinj.com/products/technology/how-can-rfid-systems-be-categorized)

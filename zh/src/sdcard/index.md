# SD 卡（SDC/MMC）

迟早有一天，你可能需要保存从传感器收集的数据、游戏文件或其他信息。最完美的选择之一就是将其存储在 SD 卡上。在本节中，我们将学习如何在 ESP32 上使用 SD 卡模块。

## MMC

MultiMediaCard（MMC）是一种早期的闪存（flash memory）存储类型，早于 SD 卡出现。它曾广泛应用于摄像机、数码相机和便携式音乐播放器等设备中。MMC 将数据以电荷形式存储在闪存单元中，这与依赖激光在反射表面上编码数据的光盘不同。

## SD（Secure Digital）卡

Secure Digital Card（SDC），通常称为 SD 卡，是 MMC 的演进版本。SD 卡被广泛用作相机、智能手机等电子设备的外部存储。一种更小的变体——microSD 卡，常用于智能手机、无人机和其他设备中。

<img style="display: block; margin: auto;" alt="SD cards" src="./images/sd-cards.png"/>
<p style="text-align: center; font-size: smaller; margin-top: 5px;">
图片来源：基于 <a href="https://en.wikipedia.org/wiki/File:SD_Cards.svg">SD card</a> by <a href="https://commons.wikimedia.org/wiki/User:Tkgd2007">Tkgd2007</a>, licensed under the GFDL and CC BY-SA 3.0, 2.5, 2.0, 1.0.
</p>

SD 卡以块（block）为单位读写数据，通常块大小为 512 字节，这使它们可以作为块设备（block device）运行；这让 SD 卡的行为非常类似于硬盘驱动器。

## 硬件需求

我们将使用 Micro SD 卡适配器模块。你可以搜索 "Micro SD Card Reader Module" 或 "Micro SD Card Adapter" 来找到它们。

<img style="width: 450px;margin: auto;display: block; " alt="Micro SD Card adapter module" src="./images/micro-sd-card-adapter-reader-module.jpg"/>

当然，你还需要一张 microSD 卡。SD 卡应格式化为 FAT32；根据你电脑的硬件情况，你可能需要一个单独的 SD 卡适配器（不是上面提到的那种）来格式化 microSD 卡。有些笔记本电脑自带 microSD 卡插槽支持。

## 协议

要与 SD 卡配合使用，我们可以使用 SD 总线协议、SPI 协议或 UHS-II 总线协议。Raspberry Pi（但不是 Raspberry Pi Pico）使用 SD 总线协议，它比 SPI 更复杂。SD 总线协议的完整细节不公开，只能通过 SD 协会获取。在本指南中，我们将使用 SPI 协议，因为我们使用的 Rust 驱动是为 SPI 设计的。

### 参考资料：

- 我强烈推荐观看 Jonathan Pallant 在 Euro Rust 2024 上关于用 Rust 编写 SD 卡驱动的[演讲](https://www.youtube.com/watch?v=-ewuFNKIAVI)。他编写了我们将要使用的驱动（最初他是为了在 ARM 上运行 MS-DOS 而创建的）。它不适用于生产系统。
- 如果你想了解 SPI 模式下的底层工作原理，可以参考这篇文章：[How to Use MMC/SDC](http://elm-chan.org/docs/mmc/mmc_e.html)

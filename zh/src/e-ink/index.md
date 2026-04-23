# 电子墨水屏（e-Ink / e-Paper）显示模块

电子墨水屏（e-Ink，也称为 e-Paper 或 electronic paper）是我见过的最迷人的显示模块之一。你可以使用微控制器（microcontroller）向显示屏写入文字或图像，然后拔掉电源——显示屏会继续显示你发送的内容，无需任何电源。真的，没有电线，没有电池，什么都没有！是的，你没听错。这就像魔法一样。

## 电子墨水屏用在哪里？

你可能以前见过电子墨水屏，甚至没有意识到。开始留意吧——你可能会发现它们无处不在！

<a href="./images/e-ink-e-paper-display-module-applications.png"><img style="display: block; margin: auto;width:80vh;" src="./images/e-ink-e-paper-display-module-applications.png" alt="e-ink applications"/></a>

电子墨水屏最著名的是用于电子阅读器（如 Amazon Kindle），其类似纸张的显示屏提供舒适的阅读体验和更长的电池续航。在零售商店中，纸质标签正在被电子墨水屏取代，使更新更加高效。你还会在公交站时刻表、机场指示牌、房间标识、智能家居设备、室内数字标牌、菜单板以及病房和护理标识中找到电子墨水屏。

## 认识硬件

有许多型号变体，尺寸和支持的颜色各不相同。有些显示屏是黑白的，而另一些提供三色或多色选项。

我们将使用 Waveshare（微雪）1.54 英寸电子墨水屏显示模块，它使用 SPI（Serial Peripheral Interface）接口。该显示屏分辨率为 200 x 200，以黑白方式显示图像。

<img style="display: block; margin: auto;width:60vh;" src="./images/e-ink-154in2-spi-display-module.png" alt="e-ink applications"/>

对于本项目，我们建议购买 1.54 英寸黑白显示模块（v2.1）。这个版本价格最实惠，对每个人都容易获得，而且与彩色型号相比，它的刷新率（refresh rate）更快。如果你使用完全相同的型号，跟着操作会更容易。

也就是说，欢迎你选择任何尺寸或颜色的显示屏；核心概念仍然相似。

## 资源

- [1.5 英寸电子纸 V2 数据手册（Datasheet）](https://files.waveshare.com/upload/e/e5/1.54inch_e-paper_V2_Datasheet.pdf)
- [1.54 英寸电子纸模块手册](https://www.waveshare.com/wiki/1.54inch_e-Paper_Module_Manual)

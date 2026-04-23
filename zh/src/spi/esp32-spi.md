# ESP32 的 SPI 外设

ESP32 包含四个 SPI 控制器，但只有其中两个：SPI2 和 SPI3 可供一般使用。它们通常被称为 HSPI 和 VSPI。另外两个 SPI 控制器 SPI0 和 SPI1 保留用于内部使用。

<img style="display: block; margin: auto;" alt="SPI 多总线多外设" src="./images/spi-multiple-spi-bus-multiple-spi-device.svg"/>

在 ESP32 上，许多 GPIO 引脚被设计为可复用于不同功能。例如，一个引脚在某个应用中可以用作通用数字输出，在另一个应用中可以用作 SPI 时钟线。这称为多路复用（multiplexing）。这意味着单个物理引脚可以连接到不同的内部信号，具体取决于你在代码中的配置方式。

ESP32 有一些 GPIO 引脚直接连接到每个 SPI 控制器。它们是：

- 对于 HSPI（SPI2）：GPIO2、GPIO4 和 GPIO12 到 GPIO15

- 对于 VSPI（SPI3）：GPIO5、GPIO18、GPIO19 和 GPIO21 到 GPIO23

这些引脚在芯片内部被布线以高效地与 SPI 配合工作，并且是常用的引脚。然而，ESP32 还允许你将 SPI 信号重新分配到许多其他 GPIO 上。

以下是 ESP32 Devkit 的默认 SPI 引脚分配：

| SPI 通道 | MOSI (SDI) | MISO (SDO) | SCLK    | CS      |
| ----------- | ---------- | ---------- | ------- | ------- |
| VSPI        | GPIO 23    | GPIO 19    | GPIO 18 | GPIO 5  |
| HSPI        | GPIO 13    | GPIO 12    | GPIO 14 | GPIO 15 |

## 在 esp-hal 中使用 SPI

要在 esp-hal crate 中使用 SPI，我们使用想要的引脚创建并配置 SPI 接口。以下是一个使用 SPI2 总线的示例。

```rust
use esp_hal::spi::master::Config as SpiConfig;
use esp_hal::spi::master::Spi;
use esp_hal::spi::Mode as SpiMode;

// 初始化 SPI
let spi_bus = Spi::new(
    peripherals.SPI2,
    SpiConfig::default()
        .with_frequency(Rate::from_mhz(60))
        .with_mode(SpiMode::_0),
)
.unwrap()
// 时钟
.with_sck(peripherals.GPIO18)
// 数据输入
.with_mosi(peripherals.GPIO23);
let cs = Output::new(peripherals.GPIO15, Level::Low, OutputConfig::default());
```

## 驱动使用示例

现在我们已经设置好了 SPI 接口，让我们使用 embedded_sdmmc crate 通过它与 SD 卡通信。我们将使用 embedded-hal-bus crate 中的 ExclusiveDevice。

```rust
use embedded_hal_bus::spi::ExclusiveDevice;
use embedded_sdmmc::SdCard;
use esp_hal::delay::Delay;

// 获取 SPI 总线的独占访问权（不共享）。
let spi_dev = ExclusiveDevice::new(spi_bus, cs, Delay).unwrap();

// 使用 SPI 设备初始化 SD 卡驱动
let sdcard = SdCard::new(spi_dev, Delay);
```

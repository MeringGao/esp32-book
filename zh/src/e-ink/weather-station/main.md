# 让乐趣开始吧

在 `main` 函数中，我们将执行通常的设置步骤，如初始化 Wi-Fi 协议栈、SPI 设备，以及创建我们之前定义的 `Dashboard` 实例。

```rust
#[esp_rtos::main]
async fn main(spawner: Spawner) -> ! {
    // generator version: 1.0.0

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    esp_alloc::heap_allocator!(#[unsafe(link_section = ".dram2_uninit")] size: 98767);

    let timg0 = TimerGroup::new(peripherals.TIMG0);
    esp_rtos::start(timg0.timer0);

    info!("Embassy initialized!");

    let radio_init = &*lib::mk_static!(
        esp_radio::Controller<'static>,
        esp_radio::init().expect("Failed to initialize Wi-Fi/BLE controller")
    );
    let rng = Rng::new();

    let stack = lib::wifi::start_wifi(radio_init, peripherals.WIFI, rng, &spawner).await;

    let spi_bus = Spi::new(
        peripherals.SPI2,
        spi::master::Config::default()
            .with_frequency(Rate::from_mhz(4))
            .with_mode(spi::Mode::_0),
    )
    .unwrap()
    //CLK
    .with_sck(peripherals.GPIO18)
    //DIN
    .with_mosi(peripherals.GPIO23);

    let cs = Output::new(peripherals.GPIO33, Level::Low, OutputConfig::default());
    let mut spi_dev = ExclusiveDevice::new(spi_bus, cs, Delay).unwrap();

    // 初始化显示屏
    // Initialize Display
    let busy_in = Input::new(
        peripherals.GPIO22,
        InputConfig::default().with_pull(Pull::None),
    );
    let dc = Output::new(peripherals.GPIO17, Level::Low, OutputConfig::default());
    let reset = Output::new(peripherals.GPIO16, Level::Low, OutputConfig::default());
    let epd = Epd1in54::new(&mut spi_dev, busy_in, dc, reset, &mut Delay, None).unwrap();

    let tls_seed = rng.random() as u64 | ((rng.random() as u64) << 32);

    let mut app = Dashboard::new(stack, epd, spi_dev);
    app.start(tls_seed).await;

    loop {
        Timer::after(Duration::from_secs(1)).await;
    }
}
```


## 克隆现有项目

你也可以克隆（或参考）我创建的项目并导航到 `wifi-webfetch` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-epaper-weather/
cd esp32-epaper-weather
```

### 如何运行？

我们需要在将程序烧录到 ESP32 时，将 Wi-Fi 名称（SSID）、Wi-Fi 密码和 Open Weather API 密钥作为环境变量传递。

```sh
SSID=YOUR_WIFI_NAME PASSWORD=YOUR_WIFI_PASSWORD API_KEY=OPEN_WEATHER_KEY  cargo run --release
```

如果一切顺利，电子纸显示屏会短暂闪烁以清除和渲染内容，你应该能看到显示的天气数据。

<img style="display: block; margin: auto;" src="../images/e-paper-weather-station.jpg"/>

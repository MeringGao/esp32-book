# 仪表盘（Dashboard）

在本章中，我们将构建主要逻辑。思路是每隔一段时间更新一次仪表盘；比如说，每 10 分钟。为此，我们将通过调用 weather 模块中的 access_website 函数来获取最新的天气数据，然后用新信息更新仪表盘的每个部分。

## 类型别名（Type Aliases）

我们将为 SPI 设备和电子纸显示屏创建类型别名，以使代码更易读和便于使用。

```rust
type SpiDevice = ExclusiveDevice<Spi<'static, esp_hal::Blocking>, Output<'static>, Delay>;
type EPD = Epd1in54<SpiDevice, Input<'static>, Output<'static>, Output<'static>, Delay>;
```

## Dashboard 结构体

我们将定义一个 Dashboard 结构体，它接收 Wi-Fi 协议栈、电子纸驱动和 SPI 设备作为输入。new 函数将初始化结构体并设置一个默认的电子纸显示屏。

```rust
pub struct Dashboard {
    display: Display1in54,
    wifi: Stack<'static>,
    epd: EPD,
    spi_dev: SpiDevice,
}

impl Dashboard {
    pub fn new(wifi: Stack<'static>, epd: EPD, spi_dev: SpiDevice) -> Self {
        Self {
            display: Display1in54::default(),
            wifi,
            epd,
            spi_dev,
        }
    }
}
```

注意：以下以 self 作为第一个参数的函数也应该包含在 impl 块内。

## 仪表盘启动

这将是启动函数，我们将在接下来的 main.rs 文件中调用它。我们还将实例化 WeatherApi。

我将显示屏旋转了 90 度。从技术上讲，你不需要这样做，因为我们使用的显示屏是 200x200 像素的。但如果你使用的是矩形显示屏，旋转它会更合理。

在一个循环中，每 10 分钟，我们将调用 refresh 函数，它会获取最新数据并更新显示屏。

```rust
pub async fn start(&mut self, tls_seed: u64) {
    self.display.set_rotation(DisplayRotation::Rotate90);

    let api = WeatherApi::new(self.wifi, tls_seed);
    loop {
        self.refresh(&api).await;

        Timer::after(Duration::from_secs(60 * 10)).await;
    }
}
```

## 用最新天气数据刷新显示屏

首先，我们通过调用 WeatherApi 的 `access_website` 函数获取最新的天气数据。然后，我们唤醒电子纸显示屏，因为我们将在最后让它进入睡眠。之后，我们清除上一帧并用白色填充显示屏。

接下来，我们绘制更新的天气信息——首先是日期，然后是天气图标和温度。之后，我们添加湿度、风速，最后是底部的签名（只是一小段 "implRust" 文字）。一旦所有内容都绘制完毕，我们就用新帧更新显示屏，等待 5 秒，然后让它进入睡眠。

```rust
pub async fn refresh(&mut self, api: &WeatherApi) {
    info!("Getting weather data");
    let weather_data = api.access_website().await;
    info!("Got weather data");

    self.epd.wake_up(&mut self.spi_dev, &mut Delay).unwrap();
    Timer::after(Duration::from_secs(5)).await;

    // 清除任何现有图像
    // Clear any existing image
    self.epd.clear_frame(&mut self.spi_dev, &mut Delay).unwrap();
    self.display.clear(Color::White).unwrap();
    self.epd
        .update_and_display_frame(&mut self.spi_dev, self.display.buffer(), &mut Delay)
        .unwrap();
    Timer::after(Duration::from_secs(5)).await;

    self.draw_date(weather_data.dt);

    self.draw_icon(weather_data.weather[0].id.icon(), Point::new(20, 50));
    self.draw_temperature(weather_data.main.temp, Point::new(20 + 70, 60));

    self.draw_humidity(weather_data.main.humidity);
    self.draw_wind(weather_data.wind.speed);

    self.draw_signature();

    self.epd
        .update_and_display_frame(&mut self.spi_dev, self.display.buffer(), &mut Delay)
        .unwrap();
    Timer::after(Duration::from_secs(5)).await;

    self.epd.sleep(&mut self.spi_dev, &mut Delay).unwrap();
}
```

## 获取图标辅助函数

这个简单的辅助函数接收图标的名称和应该绘制的位置。它将图标名称映射到对应的图像字节，然后使用 tinybmp 和 embedded_graphics 将字节转换为图像，再将其渲染到显示屏上。

```rust
fn draw_icon(&mut self, icon_name: &'static str, pos: Point) {
    let img_bytes = self.get_icon(icon_name).unwrap();

    let bmp = Bmp::from_slice(img_bytes).unwrap();
    let image = Image::new(&bmp, pos);
    image.draw(&mut self.display).unwrap();
}

pub fn get_icon(&self, icon_name: &'static str) -> Option<&'static [u8]> {
    ICONS
        .iter()
        .find(|(name, _)| *name == icon_name)
        .map(|(_, img_bytes)| *img_bytes)
}
```

## 显示温度

我们将在屏幕上显示温度，用 "°C" 后缀格式化，并在其下方绘制一条水平线。

```rust
fn draw_temperature(&mut self, temperature: f64, pos: Point) {
    let text_style = MonoTextStyle::new(&PROFONT_24_POINT, Color::Black);

    info!("Drawing temperature");
    let mut text: String<20> = String::new();
    write!(&mut text, "{}°C", temperature).unwrap();

    Text::with_baseline(&text, pos, text_style, Baseline::Top)
        .draw(&mut self.display)
        .unwrap();

    Line::new(Point::new(0, 105), Point::new(200, 105))
        .into_styled(PrimitiveStyle::with_stroke(Color::Black, 5))
        .draw(&mut self.display)
        .unwrap();
}
```

## 显示湿度

我们将在屏幕上显示湿度图标，然后在旁边显示湿度值，接着画一条竖线作为分隔。

```rust
fn draw_humidity(&mut self, humidity: i32) {
    self.draw_icon("humidity_percentage.bmp", Point::new(5, 110));

    let text_style = MonoTextStyle::new(&PROFONT_18_POINT, Color::Black);

    let mut text: String<10> = String::new();
    write!(&mut text, "{}", humidity).unwrap();

    Text::with_baseline(&text, Point::new(5 + 50, 120), text_style, Baseline::Top)
        .draw(&mut self.display)
        .unwrap();

    Line::new(Point::new(5 + 85, 120), Point::new(5 + 85, 120 + 30))
        .into_styled(PrimitiveStyle::with_stroke(Color::Black, 5))
        .draw(&mut self.display)
        .unwrap();
}
```

## 绘制风速

我们将通过首先绘制风图标来在屏幕上显示风速。然后，我们将在显示屏上显示风速值，后跟单位 "m/s"。

```rust
fn draw_wind(&mut self, wind_speed: f64) {
    self.draw_icon("air.bmp", Point::new(100, 110));

    let text_style = MonoTextStyle::new(&PROFONT_18_POINT, Color::Black);

    let mut text: String<10> = String::new();
    write!(&mut text, "{}", wind_speed).unwrap();

    Text::with_baseline(&text, Point::new(100 + 50, 120), text_style, Baseline::Top)
        .draw(&mut self.display)
        .unwrap();

    let text_style = MonoTextStyleBuilder::new()
        .font(&FONT_10X20)
        .text_color(Color::Black)
        .build();

    Text::with_baseline("m/s", Point::new(100 + 50, 140), text_style, Baseline::Top)
        .draw(&mut self.display)
        .unwrap();
}
```

## 显示日期

我们将通过在屏幕上格式化日、月、年来显示当前日期。然后，我们将在指定位置渲染文字，并在其下方绘制一条水平线作为分隔。

```rust
fn draw_date(&mut self, dt: DateTime<Utc>) {
    let text_style = MonoTextStyle::new(&PROFONT_24_POINT, Color::Black);

    let mut text: String<24> = String::new();
    write!(
        &mut text,
        "{} {} {}",
        dt.day(),
        month_name(dt.month()),
        dt.year()
    )
    .unwrap();

    Text::with_baseline(&text, Point::new(20, 10), text_style, Baseline::Top)
        .draw(&mut self.display)
        .unwrap();

    Line::new(Point::new(0, 45), Point::new(200, 45))
        .into_styled(PrimitiveStyle::with_stroke(Color::Black, 5))
        .draw(&mut self.display)
        .unwrap();
}
```

## 显示签名

这与天气无关；我们只是出于乐趣显示 "implRust"。我们计算中心位置，在中心绘制一个黑色矩形，并将文字放在正中间。

```rust
fn draw_signature(&mut self) {
    let display_width = epd1in54_v2::WIDTH as i32;
    let rect_padding = 20;

    let rect_width = display_width - 2 * rect_padding;
    let rect_height = 40;
    let rect_x = rect_padding;
    let rect_y = 170;

    let style = PrimitiveStyleBuilder::new()
        .stroke_color(Color::Black)
        .stroke_width(3)
        .fill_color(Color::Black)
        .build();

    Rectangle::new(
        Point::new(rect_x, rect_y),
        Size::new(rect_width as u32, rect_height as u32),
    )
    .into_styled(style)
    .draw(&mut self.display)
    .unwrap();

    let text = "implRust";
    let text_style = MonoTextStyle::new(&PROFONT_24_POINT, Color::White);

    let char_width = PROFONT_24_POINT.character_size.width as i32;
    let text_width = text.len() as i32 * char_width;
    let text_x = rect_x + (rect_width - text_width) / 2;

    Text::with_baseline(
        text,
        Point::new(text_x as i32, rect_y),
        text_style,
        Baseline::Top,
    )
    .draw(&mut self.display)
    .unwrap();
}
```

## 月份名称辅助函数

我们创建了一个辅助函数，根据月份数字返回缩写的月份名称。

```rust
fn month_name(month: u32) -> &'static str {
    match month {
        1 => "Jan",
        2 => "Feb",
        3 => "Mar",
        4 => "Apr",
        5 => "May",
        6 => "Jun",
        7 => "Jul",
        8 => "Aug",
        9 => "Sep",
        10 => "Oct",
        11 => "Nov",
        12 => "Dec",
        _ => "Err",
    }
}
```

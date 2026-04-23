# 天气 API（Weather API）

在本章中，我们将学习如何向 API 发送请求，并将 JSON 响应反序列化为结构体。

希望你已经在 openweathermap 网站上获取了 API 密钥。我们可以通过环境变量传递它。因此我们像这样定义常量：

```rust
const API_KEY: &str = env!("API_KEY");
```

## JSON 转结构体（JSON to Struct）

我希望你浏览 "https://openweathermap.org/current" 的 API 文档，熟悉 JSON 响应结构。在顶层，我定义了一个 WeatherData 结构体，包含 weather、main 和 wind 等字段，每个字段都使用自定义结构体类型。为了避免让教程过于冗长，我将在本章末尾包含其他结构体的定义。

```rust
#[derive(Debug, Deserialize)]
pub struct WeatherData {
    pub weather: Vec<Weather, 4>,
    pub main: Main,
    pub wind: Wind,
    #[serde(with = "chrono::serde::ts_seconds")]
    pub dt: DateTime<Utc>,
    pub name: String<20>,
}
```

在这里，我只反序列化我们需要的字段，忽略其余的 JSON 数据。

## 天气 API

让我们创建一个包装结构体来处理 API 请求。虽然一个简单的函数也可以工作，但每次都需要传递 URL 和 Wi-Fi 协议栈。通过使用结构体，我们可以实例化一次，然后在需要最新天气数据时随时调用 access_website 方法。

```rust
pub struct WeatherApi {
    wifi: Stack<'static>,
    url: String<120>,
    tls_seed: u64,
}
```

在 new 函数中，我们将通过附加 API 密钥来构建 API URL，并将其存储在字段中。

注意：如果你注意到，我使用了 "London" 作为城市名称。你可以将其替换为你的城市名称以及国家代码。参考文档以获取更多详情并相应调整。

```rust
pub fn new(wifi: Stack<'static>, tls_seed: u64) -> Self {
    let mut url = String::new();
    url.push_str(
        "https://api.openweathermap.org/data/2.5/weather?q=London&units=metric&appid=",
    )
    .unwrap();
    url.push_str(API_KEY).unwrap();
    Self {
        wifi,
        url,
        tls_seed,
    }
}
```

## 发送 API 请求

接下来，我们将实例化 DNS 套接字（socket）和 TCP 客户端。在基本设置就绪后，我们将 TCP 客户端、DNS 套接字和 TLS 配置传递给 HttpClient，并使用它向 API URL 发送请求。一旦我们收到响应，我们将使用 serde_json_core 将 JSON 载荷反序列化为 WeatherData 结构体。

```rust
pub async fn access_website(&self) -> WeatherData {
    let mut rx_buffer = [0; 4096 * 2];
    let mut tx_buffer = [0; 4096 * 2];

    let tls_config = TlsConfig::new(
        self.tls_seed,
        &mut rx_buffer,
        &mut tx_buffer,
        reqwless::client::TlsVerify::None,
    );

    let dns = DnsSocket::new(self.wifi);
    let tcp_state = TcpClientState::<1, 4096, 4096>::new();
    let tcp = TcpClient::new(self.wifi, &tcp_state);

    let mut client = HttpClient::new_with_tls(&tcp, &dns, tls_config);
    let mut buffer = [0u8; 4096];
    let mut http_req = client
        .request(reqwless::request::Method::GET, &self.url)
        .await
        .unwrap();
    let response = http_req.send(&mut buffer).await.unwrap();

    info!("Got response");
    let res = response.body().read_to_end().await.unwrap();

    let (data, _): (WeatherData, _) = serde_json_core::de::from_slice(res).unwrap();
    data
}
```

## 其余结构体

这里需要注意的一点是：API 将天气状况以数字形式给出（它也以字符串形式提供，但数字更容易处理）。我们将把这些数字映射到我们的 ConditionCode 枚举。我们还为该枚举添加了一个 icon 函数，它告诉我们每种特定天气状况应该显示哪个图标。

```rust
#[derive(Debug, Deserialize_repr)]
#[repr(u16)]
pub enum ConditionCode {
    // Group 2xx: Thunderstorm
    ThunderstormWithLightRain = 200,
    ThunderstormWithRain = 201,
    ThunderstormWithHeavyRain = 202,
    LightThunderstorm = 210,
    Thunderstorm = 211,
    HeavyThunderstorm = 212,
    RaggedThunderstorm = 221,
    ThunderstormWithLightDrizzle = 230,
    ThunderstormWithDrizzle = 231,
    ThunderstormWithHeavyDrizzle = 232,

    // Group 3xx: Drizzle
    LightIntensityDrizzle = 300,
    Drizzle = 301,
    HeavyIntensityDrizzle = 302,
    LightIntensityDrizzleRain = 310,
    DrizzleRain = 311,
    HeavyIntensityDrizzleRain = 312,
    ShowerRainAndDrizzle = 313,
    HeavyShowerRainAndDrizzle = 314,
    ShowerDrizzle = 321,

    // Group 5xx: Rain
    LightRain = 500,
    ModerateRain = 501,
    HeavyIntensityRain = 502,
    VeryHeavyRain = 503,
    ExtremeRain = 504,
    FreezingRain = 511,
    LightIntensityShowerRain = 520,
    ShowerRain = 521,
    HeavyIntensityShowerRain = 522,
    RaggedShowerRain = 531,

    // Group 6xx: Snow
    LightSnow = 600,
    Snow = 601,
    HeavySnow = 602,
    Sleet = 611,
    LightShowerSleet = 612,
    ShowerSleet = 613,
    LightRainAndSnow = 615,
    RainAndSnow = 616,
    LightShowerSnow = 620,
    ShowerSnow = 621,
    HeavyShowerSnow = 622,

    // Group 7xx: Atmosphere
    Mist = 701,
    Smoke = 711,
    Haze = 721,
    SandDustWhirls = 731,
    Fog = 741,
    Sand = 751,
    Dust = 761,
    VolcanicAsh = 762,
    Squalls = 771,
    Tornado = 781,

    // Group 800: Clear
    ClearSky = 800,

    // Group 80x: Clouds
    FewClouds = 801,
    ScatteredClouds = 802,
    BrokenClouds = 803,
    OvercastClouds = 804,
}

impl ConditionCode {
    pub fn icon(&self) -> &'static str {
        match self {
            // Thunderstorm
            ConditionCode::ThunderstormWithLightRain
            | ConditionCode::ThunderstormWithRain
            | ConditionCode::ThunderstormWithHeavyRain
            | ConditionCode::LightThunderstorm
            | ConditionCode::Thunderstorm
            | ConditionCode::HeavyThunderstorm
            | ConditionCode::RaggedThunderstorm
            | ConditionCode::ThunderstormWithLightDrizzle
            | ConditionCode::ThunderstormWithDrizzle
            | ConditionCode::ThunderstormWithHeavyDrizzle => "storm.bmp",

            // Drizzle
            ConditionCode::LightIntensityDrizzle
            | ConditionCode::Drizzle
            | ConditionCode::HeavyIntensityDrizzle
            | ConditionCode::LightIntensityDrizzleRain
            | ConditionCode::DrizzleRain
            | ConditionCode::HeavyIntensityDrizzleRain
            | ConditionCode::ShowerRainAndDrizzle
            | ConditionCode::HeavyShowerRainAndDrizzle
            | ConditionCode::ShowerDrizzle => "rainy.bmp",

            // Rain
            ConditionCode::LightRain
            | ConditionCode::ModerateRain
            | ConditionCode::HeavyIntensityRain
            | ConditionCode::VeryHeavyRain
            | ConditionCode::ExtremeRain
            | ConditionCode::LightIntensityShowerRain
            | ConditionCode::ShowerRain
            | ConditionCode::HeavyIntensityShowerRain
            | ConditionCode::RaggedShowerRain => "rainy_heavy.bmp",
            ConditionCode::FreezingRain => "weather_mix.bmp",

            // Snow
            ConditionCode::LightSnow
            | ConditionCode::Snow
            | ConditionCode::HeavySnow
            | ConditionCode::Sleet
            | ConditionCode::LightShowerSleet
            | ConditionCode::ShowerSleet
            | ConditionCode::LightRainAndSnow
            | ConditionCode::RainAndSnow
            | ConditionCode::LightShowerSnow
            | ConditionCode::ShowerSnow
            | ConditionCode::HeavyShowerSnow => "snowing.bmp",

            // Atmosphere
            ConditionCode::Mist
            | ConditionCode::Smoke
            | ConditionCode::Haze
            | ConditionCode::SandDustWhirls
            | ConditionCode::Fog
            | ConditionCode::Sand
            | ConditionCode::Dust
            | ConditionCode::VolcanicAsh
            | ConditionCode::Squalls => "foggy.bmp",
            ConditionCode::Tornado => "cyclone.bmp",

            // Clear
            ConditionCode::ClearSky => "sunny.bmp",

            // Clouds
            ConditionCode::FewClouds
            | ConditionCode::ScatteredClouds
            | ConditionCode::BrokenClouds
            | ConditionCode::OvercastClouds => "partly_cloudy_day.bmp",
        }
    }
}

#[derive(Debug, Deserialize)]
pub struct Weather {
    pub id: ConditionCode,
}

#[derive(Debug, Deserialize)]
pub struct Main {
    pub temp: f64,
    pub feels_like: f64,
    pub temp_min: f64,
    pub temp_max: f64,
    pub pressure: i32,
    pub humidity: i32,
    pub sea_level: Option<i32>,
    pub grnd_level: Option<i32>,
}

#[derive(Debug, Deserialize)]
pub struct Wind {
    pub speed: f64,
    pub deg: f64,
    pub gust: Option<f64>,
}

```

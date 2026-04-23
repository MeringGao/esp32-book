## 图标（Icons）

我们需要一组图标来表示天气、湿度、空气质量等信息。我已经将这些图标从 Google Fonts 转换为 BMP 文件，可以与 tinybmp crate 一起使用。

我们将图像字节存储在一个静态数组中，允许我们遍历它们并找到与当前天气匹配的图标。为了实现这一点，我在 `build.rs` 中添加了一个新函数，它从指定目录加载图标文件并在 `icon.rs` 中生成一个静态数组。这样，每次添加新图标时我们都不需要手动更新数组。

## 修改 build.rs 文件

在 build.rs 中，我们将添加一个扫描 `src/icons/` 目录的函数。你可以从这个仓库获取图标：https://github.com/ImplFerris/esp32-epaper-weather/tree/main/src/icons 并将它们放入你自己的 `src/icons` 文件夹中。

```rust
fn generate_icon_array() {
    let dest_path = Path::new("src/icons.rs");
    let icon_dir = Path::new("src/icons/");

    let mut generated_code = String::new();

    generated_code.push_str("pub static ICONS: &[(&str, &[u8])] = &[\n");

    if let Ok(entries) = fs::read_dir(icon_dir) {
        for entry in entries.flatten() {
            let path = entry.path();
            if let Some(file_name) = path.file_name().and_then(|f| f.to_str()) {
                generated_code.push_str(&format!(
                    "    (\"{}\", include_bytes!(\"{}\"),\n",
                    file_name, file_name
                ));
            }
        }
    }

    generated_code.push_str("];\n");

    fs::write(dest_path, generated_code).expect("Failed to write icons.rs");

    println!("cargo:rerun-if-changed={}", icon_dir.to_str().unwrap());
}
```

更新 build.rs 中的 main 函数以调用我们的新函数

```rust
fn main() {
    generate_icon_array();

    linker_be_nice();
    println!("cargo:rustc-link-arg=-Tdefmt.x");
    // make sure linkall.x is the last linker script (otherwise might cause problems with flip-link)
    println!("cargo:rustc-link-arg=-Tlinkall.x");
}
```

## icons.rs - 请勿手动修改！

如果一切顺利，你应该会在 src 文件夹中看到 icons.rs 文件，内容如下。如果文件没有创建，请尝试通过运行 `cargo build` 手动触发构建脚本。

```rust
pub static ICONS: &[(&str, &[u8])] = &[
    ("partly_cloudy_day.bmp", include_bytes!("icons/partly_cloudy_day.bmp")),
    ("nights_stay.bmp", include_bytes!("icons/nights_stay.bmp")),
    ("humidity_percentage.bmp", include_bytes!("icons/humidity_percentage.bmp")),
    ("cyclone.bmp", include_bytes!("icons/cyclone.bmp")),
    ("foggy.bmp", include_bytes!("icons/foggy.bmp")),
    ("snowing_heavy.bmp", include_bytes!("icons/snowing_heavy.bmp")),
    ("sunny.bmp", include_bytes!("icons/sunny.bmp")),
    ("cloud.bmp", include_bytes!("icons/cloud.bmp")),
    ("partly_cloudy_night.bmp", include_bytes!("icons/partly_cloudy_night.bmp")),
    ("rainy.bmp", include_bytes!("icons/rainy.bmp")),
    ("heat.bmp", include_bytes!("icons/heat.bmp")),
    ("clear_day.bmp", include_bytes!("icons/clear_day.bmp")),
    ("weather_mix.bmp", include_bytes!("icons/weather_mix.bmp")),
    ("weather_hail.bmp", include_bytes!("icons/weather_hail.bmp")),
    ("rainy_snow.bmp", include_bytes!("icons/rainy_snow.bmp")),
    ("mist.bmp", include_bytes!("icons/mist.bmp")),
    ("snowing.bmp", include_bytes!("icons/snowing.bmp")),
    ("rainy_heavy.bmp", include_bytes!("icons/rainy_heavy.bmp")),
    ("flood.bmp", include_bytes!("icons/flood.bmp")),
    ("sunny_snowing.bmp", include_bytes!("icons/sunny_snowing.bmp")),
    ("air.bmp", include_bytes!("icons/air.bmp")),
    ("storm.bmp", include_bytes!("icons/storm.bmp")),
    ("weather_snowy.bmp", include_bytes!("icons/weather_snowy.bmp")),
    ("thunderstorm.bmp", include_bytes!("icons/thunderstorm.bmp")),
    ("thermostat.bmp", include_bytes!("icons/thermostat.bmp")),
    ("rainy_light.bmp", include_bytes!("icons/rainy_light.bmp")),
];
```

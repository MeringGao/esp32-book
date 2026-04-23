## 代码

### 使用 esp-generate 生成项目

你已经在快速入门部分完成了这一步。

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 buzzer-song
```

这将打开一个屏幕，要求你选择选项。在最新的 esp-hal 中，ledc 需要我们显式启用不稳定特性（unstable features）。

- 因此请选择"启用不稳定 HAL 特性（Enable unstable HAL features）"选项。

然后按键盘上的 "s" 保存。

### 蜂鸣器引脚

我们将 GPIO 33 设置为输出引脚。这是我们连接蜂鸣器正极的引脚。

```rust
let mut buzzer = peripherals.GPIO33;
let ledc = Ledc::new(peripherals.LEDC);
```

### 歌曲实例
使用我们要播放的歌曲的速度（tempo）创建 Song 结构体的实例。
```rust
let song = Song::new(pink_panther::TEMPO);
```

### 使用 PWM 播放音符

我们将遍历歌曲的音符，使用每个音符的频率和时值。我们还为每个音符的时值添加 10% 的停顿。频率常量定义为 f64，我们将其转换为 u64 并与 Hz 一起使用。定时器和 PWM 通道配置与往常一样，只有一个区别：在定时器中，我们将频率设置为与当前音符匹配。我们将占空比设置为 50%。

```rust
 for (note, duration_type) in pink_panther::MELODY {
    // 获取音符时长
    // get music notes duration
    let note_duration = song.calc_note_duration(duration_type) as u64;
    let pause_duration = note_duration / 10; // 音符时值的 10%
    // 10% of note_duration
    if note == music::REST {
        blocking_delay(Duration::from_millis(note_duration));
        continue;
    }

    // 初始化 LEDC
    // Initialize LEDC
    let freq = Rate::from_hz(note as u32);
    let mut hstimer0 = ledc.timer::<HighSpeed>(timer::Number::Timer0);
    hstimer0
        .configure(timer::config::Config {
            duty: timer::config::Duty::Duty10Bit,
            clock_source: timer::HSClockSource::APBClk,
            frequency: freq,
        })
        .unwrap();

    // 初始化 LEDC 通道，占空比为 50%
    // Initialize LEDC Channel with duty cycle 50%
    let mut channel0 = ledc.channel(channel::Number::Channel0, buzzer.reborrow());
    channel0
        .configure(channel::config::Config {
            timer: &hstimer0,
            duty_pct: 50,
            drive_mode: DriveMode::PushPull,
        })
        .unwrap();

    blocking_delay(Duration::from_millis(note_duration - pause_duration)); // 播放 90%
    channel0.set_duty(0).unwrap();
    blocking_delay(Duration::from_millis(pause_duration)); // 停顿 10%
}
```


## 克隆现有项目
你可以克隆（或参考）我创建的项目，并导航到 `buzzer-song` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/buzzer-song
```


## 完整代码

```rust
#![no_std]
#![no_main]
#![deny(
    clippy::mem_forget,
    reason = "mem::forget is generally not safe to do with esp_hal types, especially those \
    holding buffers for the duration of a data transfer."
)]

use esp_hal::clock::CpuClock;
use esp_hal::main;
use esp_hal::time::{Duration, Instant};

use buzzer_song::{
    music::{self, Song},
    pink_panther,
};

// LEDC
use esp_hal::gpio::DriveMode;
use esp_hal::ledc::HighSpeed;
use esp_hal::ledc::timer;
use esp_hal::{
    ledc::{
        Ledc,
        channel::{self, ChannelIFace},
        timer::TimerIFace,
    },
    time::Rate,
};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

// 这会创建一个 ESP-IDF 引导加载程序所需的默认应用描述符。
// 更多信息请参阅：https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description
// This creates a default app-descriptor required by the esp-idf bootloader.
// For more information see: <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
esp_bootloader_esp_idf::esp_app_desc!();

#[main]
fn main() -> ! {
    // 生成器版本：1.0.0
    // generator version: 1.0.0

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    let mut buzzer = peripherals.GPIO33;

    let ledc = Ledc::new(peripherals.LEDC);

    let song = Song::new(pink_panther::TEMPO);

    for (note, duration_type) in pink_panther::MELODY {
        let note_duration = song.calc_note_duration(duration_type) as u64;
        let pause_duration = note_duration / 10; // 音符时值的 10%
        // 10% of note_duration
        if note == music::REST {
            blocking_delay(Duration::from_millis(note_duration));
            continue;
        }
        let freq = Rate::from_hz(note as u32);

        let mut hstimer0 = ledc.timer::<HighSpeed>(timer::Number::Timer0);
        hstimer0
            .configure(timer::config::Config {
                duty: timer::config::Duty::Duty10Bit,
                clock_source: timer::HSClockSource::APBClk,
                frequency: freq,
            })
            .unwrap();

        let mut channel0 = ledc.channel(channel::Number::Channel0, buzzer.reborrow());
        channel0
            .configure(channel::config::Config {
                timer: &hstimer0,
                duty_pct: 50,
                drive_mode: DriveMode::PushPull,
            })
            .unwrap();

        blocking_delay(Duration::from_millis(note_duration - pause_duration)); // 播放 90%

        channel0.set_duty(0).unwrap();
        blocking_delay(Duration::from_millis(pause_duration)); // 停顿 10%
    }

    loop {
        blocking_delay(Duration::from_millis(5));
    }
}

fn blocking_delay(duration: Duration) {
    let delay_start = Instant::now();
    while delay_start.elapsed() < duration {}
}
```


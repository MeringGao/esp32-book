# 如何将 ESP32 连接到 Wi-Fi 并访问网站

在本练习中，我们将把 ESP32 配置为 STA 模式以连接到您的 Wi-Fi。然后，我们将从 ESP32 向 Web 服务器发送请求，并在系统控制台中打印响应。此代码基于 esp-hal 示例。

### 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 wifi-webfetch
```

这将打开一个屏幕，要求您选择选项。为了启用 Wi-Fi，我们首先需要启用 "unstable" 和 "alloc" 功能。如果您注意到了，在选择这两个选项之前，您将无法启用 Wi-Fi 选项。因此请逐一选择：

- 首先，选择 "Enable unstable HAL features."
- 选择 "Enable allocations via the esp-alloc crate."
- 现在，您可以启用 "Enable Wi-Fi via the esp-radio crate."

还要启用日志记录功能：

- 滚动到 "Flashing, logging and debugging (espflash)" 并按 Enter。
- 然后，选择 "Use defmt to print messages"。

只需按键盘上的 "s" 保存即可。

### 更新依赖项

将以下 crate 添加到 Cargo.toml 文件中：

```toml
blocking-network-stack = { git = "https://github.com/bjoernQ/blocking-network-stack.git", rev = "b3ecefc222d8806edd221f266999ca339c52d34e", default-features = false, features = [
  "dhcpv4",
  "tcp",
] }

heapless = { version = "0.9.1" }
```

blocking-network-stack 是一个为 TCP/UDP 通信提供非异步（non-async）网络原语的 crate。

由于冲突问题，我将 embedded-io 降级到 0.6.1 版本（从 esp-generate 生成的 0.7.1 版本）。

```toml
embedded-io = { version = "0.6.1" }
```

### esp-alloc crate

"A simple no_std heap allocator for RISC-V and Xtensa processors from Espressif. Supports all currently available ESP32 devices."  esp-alloc crate 为 ESP SoC 提供了全局分配器（global allocator）。

## 初始化外设

我们将创建一个辅助函数来初始化 ESP HAL 并返回外设（peripherals）实例。我们还将使用 esp_alloc::heap_allocator! 宏设置一个 72 KiB 的堆（heap）。

```rust
fn init_hardware() -> Peripherals {
    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);
    esp_alloc::heap_allocator!(size: 72 * 1024);
    peripherals
}
```

## 初始化 Wi-Fi 控制器

我们需要向程序提供 Wi-Fi 名称和密码。我们不会在代码中硬编码这些凭据，而是从环境变量中加载它们。运行程序时，我们将一并传递环境变量。

```rust
const SSID: &str = env!("SSID");
const PASSWORD: &str = env!("PASSWORD");
```

首先，让我们初始化设置 Wi-Fi 控制器所需的 TimerGroup 和随机数生成器（Random Number Generator，RNG）。RNG 还用于初始化非异步 TCP/IP 网络协议栈（network stack），因此我们将使用 clone 以便重复使用它。

```rust
let timg0 = TimerGroup::new(peripherals.TIMG0);
let rng = Rng::new();
```

我们使用硬件定时器、RNG 和时钟外设初始化 WiFi 控制器。接下来，我们创建一个 WiFi 驱动实例来处理网络连接和管理各种接口。最后，我们将设备配置为在站点（STA）模式下运行，使其能够作为客户端连接到 WiFi 网络。

```rust
esp_rtos::start(timg0.timer0);
let radio_init = esp_radio::init().expect("Failed to initialize Wi-Fi/BLE controller");

let (mut wifi_controller, interfaces) =
    esp_radio::wifi::new(&radio_init, peripherals.WIFI, Default::default())
        .expect("Failed to initialize Wi-Fi controller");

let mut device = interfaces.sta;
```


### SocketSet 初始化

我们将创建一个 SocketSet，最多可存储 3 个套接字（socket），以便在协议栈中管理多个套接字，例如 DHCP 和 TCP。

```rust
let mut socket_set_entries: [SocketStorage; 3] = Default::default();
let mut socket_set = SocketSet::new(&mut socket_set_entries[..]);
```

### DHCP 套接字

我们将创建一个 DHCP 套接字（DHCP socket）以从 DHCP 服务器请求 IP 地址。我们可以设置传出选项，包括使用 DHCP 客户端选项 12 设置主机名（您可以在 [此处](https://efficientip.com/glossary/dhcp-option/) 阅读有关 DHCP 选项的更多信息）。我们将把 dhcp 套接字添加到我们之前创建的 SocketSet 中。

```rust
let mut dhcp_socket = smoltcp::socket::dhcpv4::Socket::new();
// 我们可以在这里设置主机名（或添加其他 DHCP 选项）
// we can set a hostname here (or add other DHCP options)
dhcp_socket.set_outgoing_options(&[DhcpOption {
    kind: 12,
    data: b"implRust",
}]);
socket_set.add(dhcp_socket);
// socket_set.add(smoltcp::socket::dhcpv4::Socket::new());
```

### 初始化网络协议栈

让我们使用网络接口和设备（这是我们之前创建的 Wi-Fi 网络接口获得的）、套接字、获取时间的闭包以及一个随机数，从 smoltcp crate 初始化 Stack。


```rust
let now = || Instant::now().duration_since_epoch().as_millis();
let mut stack = Stack::new(
    create_interface(&mut device),
    device,
    socket_set,
    now,
    rng.random(),
);
```

用于为 smoltcp TCP/IP 协议栈设置网络接口的样板函数，使用 Wi-Fi 设备（取自官方 esp-hal 示例）：

```rust
pub fn create_interface(device: &mut esp_wifi::wifi::WifiDevice) -> smoltcp::iface::Interface {
    // users could create multiple instances but since they only have one WifiDevice
    // they probably can't do anything bad with that
    smoltcp::iface::Interface::new(
        smoltcp::iface::Config::new(smoltcp::wire::HardwareAddress::Ethernet(
            smoltcp::wire::EthernetAddress::from_bytes(&device.mac_address()),
        )),
        device,
        timestamp(),
    )
}

// some smoltcp boilerplate
// 一些 smoltcp 样板代码
fn timestamp() -> smoltcp::time::Instant {
    smoltcp::time::Instant::from_micros(
        esp_hal::time::Instant::now()
            .duration_since_epoch()
            .as_micros() as i64,
    )
}
```


**Wi-Fi 操作模式**：

接下来，我们使用 ModeConfig 枚举配置 Wi-Fi 操作模式。由于我们希望 ESP32 充当 Wi-Fi 客户端，因此我们使用 Client 变体。我们需要提供 SSID（Service Set Identifier，即您的 Wi-Fi 网络名称）和 Wi-Fi 密码。

```rust
fn configure_wifi(controller: &mut WifiController<'_>) {
    controller
        .set_power_saving(esp_radio::wifi::PowerSaveMode::None)
        .unwrap();

    let client_config = ModeConfig::Client(
        ClientConfig::default()
            .with_ssid(SSID.into())
            .with_password(PASSWORD.into()),
    );
    let res = controller.set_config(&client_config);
    println!("wifi_set_configuration returned {:?}", res);

    controller.start().unwrap();
    println!("is wifi started: {:?}", controller.is_started());
}
```

## Wi-Fi 扫描并连接到接入点

接下来，我们使用默认选项执行阻塞式 Wi-Fi 网络扫描。这意味着程序将暂停并等待扫描完成。扫描完成后，它将返回可用 Wi-Fi 接入点的列表。然后我们循环遍历结果并将每个接入点的信息打印到控制台。

```rust
fn scan_wifi(controller: &mut WifiController<'_>) {
    println!("Start Wifi Scan");
    let scan_config = ScanConfig::default().with_max(10);
    let res = controller.scan_with_config(scan_config).unwrap();
    for ap in res {
        println!("{:?}", ap);
    }
}
```

## 连接 Wi-Fi

然后，我们打印控制器的支持功能，它将只显示 "Client"，因为这是我们配置的模式。

之后，我们在 Wi-Fi 控制器上调用 connect 方法。这将把 ESP32 连接到我们之前指定的 Wi-Fi 网络。

我们将在循环中持续检查 Wi-Fi 是否已连接。一旦成功连接，我们将继续执行下一步。


```rust
fn connect_wifi(controller: &mut WifiController<'_>) {
    println!("{:?}", controller.capabilities());
    println!("wifi_connect {:?}", controller.connect());

    println!("Wait to get connected");
    loop {
        match controller.is_connected() {
            Ok(true) => break,
            Ok(false) => {}
            Err(err) => panic!("{:?}", err),
        }
    }
    println!("Connected: {:?}", controller.is_connected());
}
```


### 获取 IP 地址

我们需要获取 IP 地址，因此首先等待 DHCP 过程完成。我们将持续循环，直到接口启动并已成功获取 IP 地址。

```rust
fn obtain_ip(stack: &mut Stack<'_, esp_radio::wifi::WifiDevice<'_>>) {
    println!("Wait for IP address");
    loop {
        stack.work();
        if stack.is_iface_up() {
            println!("IP acquired: {:?}", stack.get_ip_info());
            break;
        }
    }
}
```

此时，ESP32 应该已成功连接到 Wi-Fi 网络。接下来，我们将设置必要的代码以向网站发送 HTTP 请求。发送请求后，ESP32 将收到来自网站的 HTML 响应，我们将把 HTML 内容打印到控制台。


## HTTP 客户端

要向网站发送 Web 请求，我们需要一个 HTTP 客户端。为此，我们将使用 [smoltcp crate](https://crates.io/crates/smoltcp)，通过 TCP 套接字（TCP socket）发送原始 HTTP 请求。

"smoltcp 是一个独立的、事件驱动的 TCP/IP 协议栈，专为裸机（bare-metal）、实时系统设计。smoltcp 完全不需要堆分配（heap allocation），文档详尽，并且可以在稳定版 Rust 1.80 及更高版本上编译。"


### TCP 套接字

我们将使用读取和写入缓冲区初始化 TCP 套接字，以处理套接字的传入和传出数据。

```rust
let mut rx_buffer = [0u8; 1536];
let mut tx_buffer = [0u8; 1536];
let socket = stack.get_socket(&mut rx_buffer, &mut tx_buffer);

http_loop(socket)
```

## HTTP 循环 - 发送 GET 请求

让我们使用 GET 方法向网站 "www.mobile-j.de" 发送 HTTP 请求。该网站用于 esp-hal 示例，返回 "Hello fellow Rustaceans!" 以及 ferris 的 ASCII 艺术。我们将使用该网站的 IP 地址在端口 80 上打开连接，然后发送 HTTP 请求。

```rust
println!("Making HTTP request");
socket.work();

let remote_addr = IpAddress::v4(142, 250, 185, 115);
socket.open(remote_addr, 80).unwrap();
socket
    .write(b"GET / HTTP/1.0\r\nHost: www.mobile-j.de\r\n\r\n")
    .unwrap();
socket.flush().unwrap();
```

### 读取响应

接下来，我们以 20 秒超时读取服务器的响应。

```rust
let deadline = Instant::now() + Duration::from_secs(20);
let mut buffer = [0u8; 512];
while let Ok(len) = socket.read(&mut buffer) {
    // let text = unsafe { core::str::from_utf8_unchecked(&buffer[..len]) };
    let Ok(text) = core::str::from_utf8(&buffer[..len]) else {
        panic!("Invalid UTF-8 sequence encountered");
    };

    println!("{}", text);

    if Instant::now() > deadline {
        println!("Timeout");
        break;
    }
}
```

### 关闭套接字

HTTP 请求完成后，我们关闭套接字并等待 5 秒钟。

```rust
socket.disconnect();
let deadline = Instant::now() + Duration::from_secs(5);
while Instant::now() < deadline {
    socket.work();
}
```


## 克隆现有项目

您也可以克隆（或参考）我创建的项目并导航到 `wifi-webfetch` 文件夹。

```sh
git clone https://github.com/ImplFerris/esp32-projects
cd esp32-projects/wifi-webfetch
```

### 如何运行？

通常，我们只需运行 `cargo run --release`，但这次我们还需要传递 Wi-Fi 连接的环境变量。

```sh
SSID=YOUR_WIFI_NAME PASSWORD=YOUR_WIFI_PASSWORD  cargo run --release
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

use blocking_network_stack::Stack;
use embedded_io::{Read, Write};
use esp_hal::clock::CpuClock;
use esp_hal::delay::Delay;
use esp_hal::main;
use esp_hal::peripherals::Peripherals;
use esp_hal::rng::Rng;
use esp_hal::time::{Duration, Instant};
use esp_hal::timer::timg::TimerGroup;
use esp_println::{self as _, println};
use esp_radio::wifi::{ClientConfig, ModeConfig, ScanConfig, WifiController};
use smoltcp::iface::{SocketSet, SocketStorage};
use smoltcp::wire::{DhcpOption, IpAddress};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

extern crate alloc;

// This creates a default app-descriptor required by the esp-idf bootloader.
// 这会创建一个 ESP-IDF 引导加载程序所需的应用描述符。
// For more information see: <https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/app_image_format.html#application-description>
// 更多信息请参阅上述链接。
esp_bootloader_esp_idf::esp_app_desc!();

const SSID: &str = env!("SSID");
const PASSWORD: &str = env!("PASSWORD");

#[main]
fn main() -> ! {
    // generator version: 1.0.0

    let peripherals = init_hardware();

    esp_alloc::heap_allocator!(#[unsafe(link_section = ".dram2_uninit")] size: 98767);

    let timg0 = TimerGroup::new(peripherals.TIMG0);
    let rng = Rng::new();

    esp_rtos::start(timg0.timer0);
    let radio_init = esp_radio::init().expect("Failed to initialize Wi-Fi/BLE controller");

    let (mut wifi_controller, interfaces) =
        esp_radio::wifi::new(&radio_init, peripherals.WIFI, Default::default())
            .expect("Failed to initialize Wi-Fi controller");

    let mut device = interfaces.sta;

    // let mut stack = setup_network_stack(device, &mut rng);
    let mut socket_set_entries: [SocketStorage; 3] = Default::default();
    let mut socket_set = SocketSet::new(&mut socket_set_entries[..]);
    let mut dhcp_socket = smoltcp::socket::dhcpv4::Socket::new();

    // 我们可以在这里设置主机名（或添加其他 DHCP 选项）
    // we can set a hostname here (or add other DHCP options)
    dhcp_socket.set_outgoing_options(&[DhcpOption {
        kind: 12,
        data: b"implRust",
    }]);
    socket_set.add(dhcp_socket);
    // sta_socket_set.add(smoltcp::socket::dhcpv4::Socket::new());

    let now = || Instant::now().duration_since_epoch().as_millis();
    let mut stack = Stack::new(
        create_interface(&mut device),
        device,
        socket_set,
        now,
        rng.random(),
    );

    configure_wifi(&mut wifi_controller);
    scan_wifi(&mut wifi_controller);
    connect_wifi(&mut wifi_controller);
    obtain_ip(&mut stack);

    let mut rx_buffer = [0u8; 1536];
    let mut tx_buffer = [0u8; 1536];
    let socket = stack.get_socket(&mut rx_buffer, &mut tx_buffer);

    http_loop(socket)
}

pub fn create_interface(device: &mut esp_radio::wifi::WifiDevice) -> smoltcp::iface::Interface {
    // users could create multiple instances but since they only have one WifiDevice
    // they probably can't do anything bad with that
    smoltcp::iface::Interface::new(
        smoltcp::iface::Config::new(smoltcp::wire::HardwareAddress::Ethernet(
            smoltcp::wire::EthernetAddress::from_bytes(&device.mac_address()),
        )),
        device,
        timestamp(),
    )
}

// some smoltcp boilerplate
// 一些 smoltcp 样板代码
fn timestamp() -> smoltcp::time::Instant {
    smoltcp::time::Instant::from_micros(
        esp_hal::time::Instant::now()
            .duration_since_epoch()
            .as_micros() as i64,
    )
}

fn init_hardware() -> Peripherals {
    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);
    esp_alloc::heap_allocator!(size: 72 * 1024);
    peripherals
}

fn configure_wifi(controller: &mut WifiController<'_>) {
    controller
        .set_power_saving(esp_radio::wifi::PowerSaveMode::None)
        .unwrap();

    let client_config = ModeConfig::Client(
        ClientConfig::default()
            .with_ssid(SSID.into())
            .with_password(PASSWORD.into()),
    );
    let res = controller.set_config(&client_config);
    println!("wifi_set_configuration returned {:?}", res);

    controller.start().unwrap();
    println!("is wifi started: {:?}", controller.is_started());
}

fn scan_wifi(controller: &mut WifiController<'_>) {
    println!("Start Wifi Scan");
    let scan_config = ScanConfig::default().with_max(10);
    let res = controller.scan_with_config(scan_config).unwrap();
    for ap in res {
        println!("{:?}", ap);
    }
}

fn connect_wifi(controller: &mut WifiController<'_>) {
    println!("{:?}", controller.capabilities());
    println!("wifi_connect {:?}", controller.connect());

    println!("Wait to get connected");
    loop {
        match controller.is_connected() {
            Ok(true) => break,
            Ok(false) => {}
            Err(err) => panic!("{:?}", err),
        }
    }
    println!("Connected: {:?}", controller.is_connected());
}

fn obtain_ip(stack: &mut Stack<'_, esp_radio::wifi::WifiDevice<'_>>) {
    println!("Wait for IP address");
    loop {
        stack.work();
        if stack.is_iface_up() {
            println!("IP acquired: {:?}", stack.get_ip_info());
            break;
        }
    }
}

fn http_loop(
    mut socket: blocking_network_stack::Socket<'_, '_, esp_radio::wifi::WifiDevice<'_>>,
) -> ! {
    println!("Starting HTTP client loop");
    let delay = Delay::new();
    loop {
        println!("Making HTTP request");
        socket.work();

        let remote_addr = IpAddress::v4(142, 250, 185, 115);
        socket.open(remote_addr, 80).unwrap();
        socket
            .write(b"GET / HTTP/1.0\r\nHost: www.mobile-j.de\r\n\r\n")
            .unwrap();
        socket.flush().unwrap();

        let deadline = Instant::now() + Duration::from_secs(20);
        let mut buffer = [0u8; 512];
        while let Ok(len) = socket.read(&mut buffer) {
            // let text = unsafe { core::str::from_utf8_unchecked(&buffer[..len]) };
            let Ok(text) = core::str::from_utf8(&buffer[..len]) else {
                panic!("Invalid UTF-8 sequence encountered");
            };

            println!("{}", text);

            if Instant::now() > deadline {
                println!("Timeout");
                break;
            }
        }

        socket.disconnect();
        let deadline = Instant::now() + Duration::from_secs(5);
        while Instant::now() < deadline {
            socket.work();
        }

        delay.delay_millis(1000);
    }
}
```

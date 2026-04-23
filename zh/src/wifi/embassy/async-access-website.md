# 使用 Embassy 异步连接 ESP32 到 Wi-Fi 并访问网站

在本练习中，我们将再次使用 STA 模式访问网站。但这一次，我们将以异步（asynchronously）方式进行。我们将使用 reqwless crate 作为我们的 HTTP 客户端，并使用 Embassy 框架为嵌入式环境启用异步功能。除此之外，我们还将包含额外的辅助 crate。


## 使用 esp-generate 生成项目

要创建项目，请使用 `esp-generate` 命令。运行以下命令：

```sh
esp-generate --chip esp32 wifi-async-http
```

这将打开一个屏幕，要求您选择选项。为了启用 Wi-Fi，我们首先需要启用 "unstable" 和 "alloc" 功能。如果您注意到了，在选择这两个选项之前，您将无法启用 Wi-Fi 选项。因此请逐一选择：

- 首先，选择 "Enable unstable HAL features."
- 选择 "Enable allocations via the esp-alloc crate."
- 现在，您可以启用 "Enable Wi-Fi via esp-radio crate."
- 选择 "Adds embassy framework support"。

还要启用日志记录功能：

- 滚动到 "Flashing, logging and debugging (espflash)" 并按 Enter。
- 然后，选择 "Use defmt to print messages"。

只需按键盘上的 "s" 保存即可。

## 更新依赖项

### Embassy net

embassy-net 构建在 smoltcp crate 之上，为嵌入式系统提供异步网络协议栈（async network stack）。它提供更高层次的 API，并支持 IPv4、IPv6、以太网、裸 IP 介质、TCP、UDP、DNS、DHCPv4 和多播。

当您使用 esp-generate 生成项目时选择 Wi-Fi 和 Embassy 选项，此 crate 会自动添加。但是，我们需要一个额外的功能："dns"。这对于执行 HTTP 请求是必要的，因为网站名称（例如 google.com）需要解析为 IP 地址。请记住，在之前的练习中，我们手动提供了网站的 IP 地址。

```toml
# 在 Cargo.toml 中更新 embassy-net
# Updated embassy-net in Cargo.toml
embassy-net = { version = "0.7.1", features = [
  "defmt",
  "dhcpv4",
  "medium-ethernet",
  "tcp",
  "udp",
  # 新增：
  #addition:
  "dns",
] }
```

### smoltcp

smoltcp crate 也会被添加到 Cargo.toml 中。我们需要添加 "dns-max-server-count-4" 功能才能使用 DNS 服务器。

```toml
# 在 Cargo.toml 中更新 smoltcp
# Updated smoltcp in Cargo.toml
smoltcp = { version = "0.12.0", default-features = false, features = [
    "defmt",
    "medium-ethernet",
    "multicast",
    "proto-dhcpv4",
    "proto-dns",
    "proto-ipv4",
    "socket-dns",
    "socket-icmp",
    "socket-raw",
    "socket-tcp",
    "socket-udp",
    # 新增：
    # addition:
    "dns-max-server-count-4",
] }
```

### Reqwless
reqwless crate 是一个用于嵌入式系统的 HTTP 客户端，可与任何实现了 embedded-io crate 中 trait 的传输层配合使用。

```toml
reqwless = { version = "0.13.0", default-features = false, features = [
    "embedded-tls",
] }
```

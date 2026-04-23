# 应用启动流程

在进入烧录部分之前，我们的代码中还有一行需要添加。为了理解原因，让我们看看微控制器上电时会发生什么。在 Espressif 芯片上，我们的程序并不是第一个运行的。有两个引导加载程序（bootloader）在我们的应用程序启动之前执行。

<img style="display: block; margin: auto;" alt="ESP32 Bootloader and App Image Descriptor" src="./images/esp32-bootloader-flow.svg"/>

第一阶段引导加载程序（First Stage Bootloader）在芯片制造时被烧录到 ROM 中，你无法更改它。第二阶段引导加载程序（Second Stage Bootloader）才是真正加载你的应用程序并设置内存的部分。使用的标准第二阶段引导加载程序是 ESP-IDF Bootloader。

默认情况下，当你使用 `espflash` 烧录程序时，它会自动包含一个用默认设置预编译好的 ESP-IDF 引导加载程序，所以你不需要自己构建或配置它。

## 应用描述符（Application Descriptor）

ESP-IDF 引导加载程序在启动过程中会读取一个名为"应用描述符（Application Descriptor）"的段来验证你的固件（firmware）。这个应用描述符包含关于你应用程序的元数据：版本信息、项目名称、编译时间和日期、ESP-IDF 版本，以及应用程序 ELF 文件的 SHA256 哈希值。

我们不需要自己创建这个描述符。你可以添加以下 crate：

```toml
esp-bootloader-esp-idf = { version = "0.4.0", features = ["esp32"] }
```

然后将这个宏添加到你的 `main.rs` 文件中：

```rust
esp_bootloader_esp_idf::esp_app_desc!();
```

这个宏会自动生成 ESP-IDF 引导加载程序期望在每个固件镜像中找到的应用描述符结构。

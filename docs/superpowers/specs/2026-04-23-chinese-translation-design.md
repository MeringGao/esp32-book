# ESP32-Book 中文翻译设计方案

## 项目背景

将 mdBook 项目 **"impl Rust for ESP32"**（约 172 个 Markdown 文件、21,000 行内容）翻译为中文版，在现有仓库内建立双语版本，保持英文原文不变。

## 设计决策

### 1. 双语结构（方案 A）

- `src/` — 英文原文（现有，完全不变）
- `zh/src/` — 中文译文（新建，镜像 `src/` 结构）
- `book.toml` — 英文版配置（现有）
- `book-zh.toml` — 中文版配置（新建）

构建命令：
- 英文：`mdbook build`
- 中文：`mdbook build --config book-zh.toml`

### 2. 翻译范围

- **正文 prose**：完整翻译为中文
- **代码注释**：双语格式（中文注释在前，英文原文在后）
- **章节标题**：翻译为中文（`zh/src/SUMMARY.md`）
- **图片**：保留原样，不处理
- **代码块**：保留原样，仅注释双语化
- **链接/引用**：保持原有相对路径，确保图片等资源正常加载

### 3. 术语规范

#### 保留英文（不翻译）

| 类别 | 示例 |
|------|------|
| Rust 关键字/特性 | `no_std`, `std`, `no_main` |
| 框架/库名 | `Embassy`, `ratatui`, `mipidsi`, `trouble` |
| 缩写（标准用法） | `GPIO`, `PWM`, `ADC`, `I2C`, `SPI`, `BLE`, `GATT`, `GAP`, `HAL`, `PAC` |
| 硬件型号 | `ESP32`, `ESP32-S3`, `WROOM`, `DevKit V1` |
| 工具名 | `Rust`, `Cargo`, `rustc`, `espflash` |

#### 翻译并标注英文（首次出现）

格式：`中文（English Term）`

| 英文 | 中文 |
|------|------|
| peripheral | 外设（Peripheral） |
| toolchain | 工具链（Toolchain） |
| cross compilation | 交叉编译（Cross Compilation） |
| bare metal | 裸机（Bare Metal） |
| linker script | 链接器脚本（Linker Script） |
| flash (v.) | 烧录 / 刷写（Flash） |
| firmware | 固件（Firmware） |
| interrupt | 中断（Interrupt） |
| register | 寄存器（Register） |
| datasheet | 数据手册（Datasheet） |
| breadboard | 面包板（Breadboard） |
| pinout | 引脚图（Pinout） |

缩写首次出现展开全称：硬件抽象层（Hardware Abstraction Layer, HAL）

#### 代码注释双语格式

```rust
// 开启 LED
// Turn on the LED
led.set_high();
```

### 4. 通用风格

- 人称：保持原书友好口吻，"我们"引导读者实验
- 技术概念：首次出现时铺垫解释，标注英文原文
- 章节标题：简洁直译

## 工作流程

### 阶段 1：术语表提取

扫描 `src/` 所有 Markdown，提取高频技术术语，建立完整术语表文件。

### 阶段 2：结构搭建

1. 创建 `book-zh.toml`
2. 创建 `zh/src/` 目录（镜像 `src/`）
3. 翻译 `zh/src/SUMMARY.md`

### 阶段 3：批量翻译（按章节分组）

1. **基础篇**：index.md, esp32-intro/, dev-env.md, quick-start.md, abstraction-layers.md, faq.md, help.md
2. **核心概念篇**：std-to-no-std/, core-concepts/pwm/, core-concepts/adc/, core-concepts/voltage-divider.md
3. **外设实战篇**：led/, buzzer/, ultrasonic/, pir-sensor/, servo/, ldr/, wifi/, i2c/, oled/, thermistor/, spi/, sdcard/, rfid/, joystick/
4. **高级篇**：bluetooth/, lcd/, e-ink/, tft-display/, ratatui/, embassy/
5. **收尾**：projects.md

### 阶段 4：验证

1. `mdbook build --config book-zh.toml` 确保构建成功
2. 检查 broken link 或格式错误
3. 抽查渲染效果

## 边界与约束

- 忠于原书内容，不增删技术细节
- 书中硬件基于 ESP32 DevKit V1，翻译时不擅自适配到 ESP32-S3（用户自行设备差异在审校时处理）
- 图片中的英文不翻译（因是静态图片资源）
- 代码中的 crate 名、函数名、变量名保留英文

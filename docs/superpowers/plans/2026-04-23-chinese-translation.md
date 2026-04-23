# ESP32-Book 中文翻译实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将 "impl Rust for ESP32" mdBook 完整翻译为中文版，建立双语仓库结构。

**Architecture:** 保持 `src/` 英文原文不变，新建 `zh/src/` 镜像目录存放中文译文，`book-zh.toml` 用于构建中文版。按章节分组逐批翻译，统一术语表确保一致性。

**Tech Stack:** mdBook, Markdown

---

## 文件结构映射

```
esp32-book/
├── book.toml                    # 英文版配置（不变）
├── book-zh.toml                 # 中文版配置（新建）
├── src/                         # 英文原文（不变）
│   ├── SUMMARY.md
│   ├── index.md
│   └── ...
└── zh/
    └── src/                     # 中文译文（新建，镜像 src/ 结构）
        ├── SUMMARY.md
        ├── index.md
        └── ...
```

---

### Task 1: 提取并建立术语表

**Files:**
- Create: `zh/TERMINOLOGY.md`

**Context:** 先扫描全书提取高频技术术语，建立统一术语表供后续所有翻译任务参考。

- [ ] **Step 1: 扫描全书高频术语**

阅读 `src/` 下所有 Markdown 文件，重点关注：
- 技术概念词（peripheral, toolchain, cross compilation, bare metal 等）
- 硬件/协议缩写（GPIO, PWM, ADC, I2C, SPI, BLE, GATT, GAP 等）
- Rust/嵌入式专有名词（HAL, PAC, no_std, Embassy 等）
- 常用动词（flash, configure, initialize 等）

- [ ] **Step 2: 建立术语表文件**

创建 `zh/TERMINOLOGY.md`，格式如下：

```markdown
# 术语表

## 保留英文（不翻译）

| 术语 | 说明 |
|------|------|
| no_std | Rust 特性 |
| Embassy | 异步框架 |
| GPIO | 通用输入输出 |
| ... | ... |

## 翻译并标注英文（首次出现）

| 英文 | 中文 | 备注 |
|------|------|------|
| peripheral | 外设 | 通用 |
| toolchain | 工具链 | 通用 |
| ... | ... | ... |
```

- [ ] **Step 3: 提交术语表**

```bash
git add zh/TERMINOLOGY.md
git commit -m "docs: add Chinese translation terminology table"
```

---

### Task 2: 搭建中文版本结构

**Files:**
- Create: `book-zh.toml`
- Create: `zh/src/SUMMARY.md`
- Create: `zh/src/` 目录树（镜像 `src/` 的空目录结构）

- [ ] **Step 1: 创建 book-zh.toml**

```toml
[book]
title = "impl Rust for ESP32（中文版）"
authors = ["ImplFerris", "中文译者"]
language = "zh"
src = "zh/src"

[output.html]
git-repository-url = "https://github.com/ImplFerris/esp32-book"
cname = "esp32.implrust.com"
preferred-dark-theme = "ayu"
default-theme = "dark"
mathjax-support = true
sidebar-header-nav = false
additional-css = ["theme/css/custom.css"]

[output.html.fold]
enable = true
level = 1
```

- [ ] **Step 2: 创建 zh/src 目录结构**

```bash
cd /home/mering/projects/esp32-book
# 镜像 src 的目录结构到 zh/src
find src -type d | sed 's|^src|zh/src|' | xargs mkdir -p
```

- [ ] **Step 3: 翻译 SUMMARY.md**

将 `src/SUMMARY.md` 翻译为 `zh/src/SUMMARY.md`：
- 章节标题翻译为中文
- 文件路径保持与 `src/` 镜像一致（如 `./index.md`）
- 注释掉的章节保持注释状态

- [ ] **Step 4: 验证目录结构**

```bash
# 确认目录结构镜像成功
find zh/src -type d | sort
find src -type d | sed 's|^src|zh/src|' | sort | diff - <(find zh/src -type d | sort)
# Expected: 无输出（结构完全一致）
```

- [ ] **Step 5: 提交结构搭建**

```bash
git add book-zh.toml zh/src/SUMMARY.md
git add zh/src/  # 空目录可能不会被 git 跟踪，后续翻译文件会自然建立目录
git commit -m "feat: setup Chinese translation structure (book-zh.toml + zh/src/ + SUMMARY)"
```

---

### Task 3: 翻译基础篇（Introduction + 环境）

**Files:**
- Create: `zh/src/index.md`
- Create: `zh/src/esp32-intro/esp32-family.md`
- Create: `zh/src/esp32-intro/pinout.md`
- Create: `zh/src/dev-env.md`
- Create: `zh/src/quick-start.md`
- Create: `zh/src/abstraction-layers.md`
- Create: `zh/src/faq.md`
- Create: `zh/src/help.md`

**Translation rules:**
- 正文完整翻译为中文
- 代码注释双语格式：`// 中文注释\n// Original English comment`
- 术语首次出现标注英文：`外设（Peripheral）`
- 保留所有原始 Markdown 格式、链接、图片引用
- 保留代码块内容不变，仅翻译注释

- [ ] **Step 1: 翻译 index.md**
- [ ] **Step 2: 翻译 esp32-intro/esp32-family.md**
- [ ] **Step 3: 翻译 esp32-intro/pinout.md**
- [ ] **Step 4: 翻译 dev-env.md**
- [ ] **Step 5: 翻译 quick-start.md**
- [ ] **Step 6: 翻译 abstraction-layers.md**
- [ ] **Step 7: 翻译 faq.md**
- [ ] **Step 8: 翻译 help.md**
- [ ] **Step 9: 提交基础篇**

```bash
git add zh/src/index.md zh/src/esp32-intro/ zh/src/dev-env.md zh/src/quick-start.md zh/src/abstraction-layers.md zh/src/faq.md zh/src/help.md
git commit -m "feat(zh): translate introduction and environment chapters"
```

---

### Task 4: 翻译核心概念篇 Part 1（std-to-no-std 基础）

**Files:**
- Create: `zh/src/std-to-no-std/index.md`
- Create: `zh/src/std-to-no-std/cross-compilation/index.md`
- Create: `zh/src/std-to-no-std/cross-compilation/embedded.md`
- Create: `zh/src/std-to-no-std/cross-compilation/toolchain.md`
- Create: `zh/src/std-to-no-std/no-std.md`
- Create: `zh/src/std-to-no-std/panic-handler.md`
- Create: `zh/src/std-to-no-std/no-main.md`
- Create: `zh/src/std-to-no-std/peripherals.md`
- Create: `zh/src/std-to-no-std/init-esp-hal.md`
- Create: `zh/src/std-to-no-std/led.md`
- Create: `zh/src/std-to-no-std/linker-script.md`
- Create: `zh/src/std-to-no-std/esp-idf-app-descriptor.md`
- Create: `zh/src/std-to-no-std/flash.md`
- Create: `zh/src/std-to-no-std/delay.md`

- [ ] **Step 1-13: 逐文件翻译（按上述列表顺序）**
- [ ] **Step 14: 提交**

```bash
git add zh/src/std-to-no-std/
git commit -m "feat(zh): translate std-to-no-std core concepts"
```

---

### Task 5: 翻译核心概念篇 Part 2（PWM + ADC + Voltage Divider）

**Files:**
- Create: `zh/src/core-concepts/index.md`
- Create: `zh/src/core-concepts/pwm/index.md`
- Create: `zh/src/core-concepts/pwm/pwm-in-depth.md`
- Create: `zh/src/core-concepts/pwm/led-pwm-controller.md`
- Create: `zh/src/core-concepts/pwm/mcpwm.md`
- Create: `zh/src/core-concepts/adc/index.md`
- Create: `zh/src/core-concepts/adc/adc-in-esp32.md`
- Create: `zh/src/core-concepts/voltage-divider.md`

- [ ] **Step 1-7: 逐文件翻译**
- [ ] **Step 8: 提交**

```bash
git add zh/src/core-concepts/
git commit -m "feat(zh): translate PWM, ADC and voltage divider concepts"
```

---

### Task 6: 翻译 LED + Buzzer 章节

**Files:**
- Create: `zh/src/led/index.md`
- Create: `zh/src/led/code.md`
- Create: `zh/src/led/external-led.md`
- Create: `zh/src/buzzer/index.md`
- Create: `zh/src/buzzer/circuit.md`
- Create: `zh/src/buzzer/active-beep.md`
- Create: `zh/src/buzzer/play-songs/index.md`
- Create: `zh/src/buzzer/play-songs/music-theory.md`
- Create: `zh/src/buzzer/play-songs/music-module.md`
- Create: `zh/src/buzzer/play-songs/song-module.md`
- Create: `zh/src/buzzer/play-songs/code.md`

- [ ] **Step 1-10: 逐文件翻译**
- [ ] **Step 11: 提交**

```bash
git add zh/src/led/ zh/src/buzzer/
git commit -m "feat(zh): translate LED and buzzer chapters"
```

---

### Task 7: 翻译 Ultrasonic + PIR + Servo 章节

**Files:**
- Create: `zh/src/ultrasonic/index.md`
- Create: `zh/src/ultrasonic/how-it-works.md`
- Create: `zh/src/ultrasonic/circuit.md`
- Create: `zh/src/ultrasonic/code.md`
- Create: `zh/src/ultrasonic/using-buzzer.md`
- Create: `zh/src/pir-sensor/index.md`
- Create: `zh/src/pir-sensor/settings.md`
- Create: `zh/src/pir-sensor/circuit.md`
- Create: `zh/src/pir-sensor/code.md`
- Create: `zh/src/pir-sensor/burglar-alarm.md`
- Create: `zh/src/servo/index.md`
- Create: `zh/src/servo/pwm.md`
- Create: `zh/src/servo/circuit.md`
- Create: `zh/src/servo/ledc.md`
- Create: `zh/src/servo/code.md`
- Create: `zh/src/servo/mcpwm.md`

- [ ] **Step 1-15: 逐文件翻译**
- [ ] **Step 16: 提交**

```bash
git add zh/src/ultrasonic/ zh/src/pir-sensor/ zh/src/servo/
git commit -m "feat(zh): translate ultrasonic, PIR and servo chapters"
```

---

### Task 8: 翻译 ADC + LDR 章节

**Files:**
- Create: `zh/src/ldr/index.md`
- Create: `zh/src/ldr/how-it-works.md`
- Create: `zh/src/ldr/led/index.md`
- Create: `zh/src/ldr/led/code.md`

- [ ] **Step 1-3: 逐文件翻译**
- [ ] **Step 4: 提交**

```bash
git add zh/src/ldr/
git commit -m "feat(zh): translate LDR chapter"
```

---

### Task 9: 翻译 Wi-Fi 章节 Part 1（基础 + Station）

**Files:**
- Create: `zh/src/wifi/index.md`
- Create: `zh/src/wifi/sta-mode-access-website.md`
- Create: `zh/src/wifi/static-ip.md`
- Create: `zh/src/wifi/led/index.md`
- Create: `zh/src/wifi/led/webpage-control-led.md`

- [ ] **Step 1-4: 逐文件翻译**
- [ ] **Step 5: 提交**

```bash
git add zh/src/wifi/index.md zh/src/wifi/sta-mode-access-website.md zh/src/wifi/static-ip.md zh/src/wifi/led/
git commit -m "feat(zh): translate Wi-Fi basics and station mode"
```

---

### Task 10: 翻译 Wi-Fi 章节 Part 2（Embassy + Web Server + AP）

**Files:**
- Create: `zh/src/wifi/embassy/async-access-website.md`
- Create: `zh/src/wifi/embassy/connecting-wifi.md`
- Create: `zh/src/wifi/embassy/http-request.md`
- Create: `zh/src/wifi/web-server/index.md`
- Create: `zh/src/wifi/web-server/wifi.md`
- Create: `zh/src/wifi/web-server/serve-website.md`
- Create: `zh/src/wifi/web-server/exposing-to-internet.md`
- Create: `zh/src/wifi/access-point/index.md`
- Create: `zh/src/wifi/access-point/running.md`

- [ ] **Step 1-8: 逐文件翻译**
- [ ] **Step 9: 提交**

```bash
git add zh/src/wifi/embassy/ zh/src/wifi/web-server/ zh/src/wifi/access-point/
git commit -m "feat(zh): translate Wi-Fi embassy, web server and access point"
```

---

### Task 11: 翻译 I2C + OLED 章节

**Files:**
- Create: `zh/src/i2c/index.md`
- Create: `zh/src/i2c/i2c-and-rust.md`
- Create: `zh/src/i2c/esp32-i2c.md`
- Create: `zh/src/oled/index.md`
- Create: `zh/src/oled/how-it-works.md`
- Create: `zh/src/oled/circuit.md`
- Create: `zh/src/oled/crates.md`
- Create: `zh/src/oled/hello-rust/index.md`
- Create: `zh/src/oled/draw-image/index.md`
- Create: `zh/src/oled/draw-image/code.md`
- Create: `zh/src/oled/draw-image/multi-byte.md`
- Create: `zh/src/oled/draw-image/multi-byte-code.md`
- Create: `zh/src/oled/draw-image/bitmap.md`

- [ ] **Step 1-12: 逐文件翻译**
- [ ] **Step 13: 提交**

```bash
git add zh/src/i2c/ zh/src/oled/
git commit -m "feat(zh): translate I2C and OLED chapters"
```

---

### Task 12: 翻译 Thermistor + SPI + SD Card 章节

**Files:**
- Create: `zh/src/thermistor/index.md`
- Create: `zh/src/thermistor/voltage-divider.md`
- Create: `zh/src/thermistor/adc.md`
- Create: `zh/src/thermistor/adc-maths.md`
- Create: `zh/src/thermistor/esp32-non-linear.md`
- Create: `zh/src/thermistor/non-linear.md`
- Create: `zh/src/thermistor/b-equation.md`
- Create: `zh/src/thermistor/steinhart.md`
- Create: `zh/src/thermistor/circuit.md`
- Create: `zh/src/thermistor/print-adc.md`
- Create: `zh/src/thermistor/oled/index.md`
- Create: `zh/src/thermistor/oled/code.md`
- Create: `zh/src/spi/index.md`
- Create: `zh/src/spi/spi-and-rust.md`
- Create: `zh/src/spi/esp32-spi.md`
- Create: `zh/src/sdcard/index.md`
- Create: `zh/src/sdcard/circuit.md`
- Create: `zh/src/sdcard/read-sdcard.md`
- Create: `zh/src/sdcard/write-sdcard.md`

- [ ] **Step 1-18: 逐文件翻译**
- [ ] **Step 19: 提交**

```bash
git add zh/src/thermistor/ zh/src/spi/ zh/src/sdcard/
git commit -m "feat(zh): translate thermistor, SPI and SD card chapters"
```

---

### Task 13: 翻译 RFID 章节

**Files:**
- Create: `zh/src/rfid/index.md`
- Create: `zh/src/rfid/rc522.md`
- Create: `zh/src/rfid/mifare.md`
- Create: `zh/src/rfid/flow.md`
- Create: `zh/src/rfid/circuit.md`
- Create: `zh/src/rfid/read-uid.md`
- Create: `zh/src/rfid/read-data.md`
- Create: `zh/src/rfid/dump-memory.md`
- Create: `zh/src/rfid/access-bits.md`
- Create: `zh/src/rfid/access-bits-calculator.md`
- Create: `zh/src/rfid/write-data.md`
- Create: `zh/src/rfid/change-auth-key.md`
- Create: `zh/src/rfid/project-ideas.md`

- [ ] **Step 1-12: 逐文件翻译**
- [ ] **Step 13: 提交**

```bash
git add zh/src/rfid/
git commit -m "feat(zh): translate RFID chapter"
```

---

### Task 14: 翻译 Joystick + Bluetooth 章节

**Files:**
- Create: `zh/src/joystick/index.md`
- Create: `zh/src/joystick/movement-and-12-bit-adc-value.md`
- Create: `zh/src/joystick/pin-layout.md`
- Create: `zh/src/joystick/circuit.md`
- Create: `zh/src/joystick/print-adc-values.md`
- Create: `zh/src/bluetooth/index.md`
- Create: `zh/src/bluetooth/ble/index.md`
- Create: `zh/src/bluetooth/ble/gap.md`
- Create: `zh/src/bluetooth/ble/gatt.md`
- Create: `zh/src/bluetooth/trouble/index.md`
- Create: `zh/src/bluetooth/trouble/ble-module.md`
- Create: `zh/src/bluetooth/trouble/running-ble-stack.md`
- Create: `zh/src/bluetooth/trouble/advertise.md`
- Create: `zh/src/bluetooth/trouble/gatt-events.md`
- Create: `zh/src/bluetooth/trouble/notifier.md`
- Create: `zh/src/bluetooth/trouble/connect.md`

- [ ] **Step 1-15: 逐文件翻译**
- [ ] **Step 16: 提交**

```bash
git add zh/src/joystick/ zh/src/bluetooth/
git commit -m "feat(zh): translate joystick and bluetooth chapters"
```

---

### Task 15: 翻译 LCD 章节

**Files:**
- Create: `zh/src/lcd/index.md`
- Create: `zh/src/lcd/how-it-works.md`
- Create: `zh/src/lcd/pin-layout.md`
- Create: `zh/src/lcd/circuit.md`
- Create: `zh/src/lcd/hello-rust.md`
- Create: `zh/src/lcd/supported-characters.md`
- Create: `zh/src/lcd/custom-chars.md`
- Create: `zh/src/lcd/lcd-custom-char-generator.md`
- Create: `zh/src/lcd/display-custom-chars.md`
- Create: `zh/src/lcd/multi-custom-gen.md`
- Create: `zh/src/lcd/multi-custom-character.md`
- Create: `zh/src/lcd/custom-symbols-index.md`

- [ ] **Step 1-11: 逐文件翻译**
- [ ] **Step 12: 提交**

```bash
git add zh/src/lcd/
git commit -m "feat(zh): translate LCD chapter"
```

---

### Task 16: 翻译 E-ink + TFT + Ratatui 章节

**Files:**
- Create: `zh/src/e-ink/index.md`
- Create: `zh/src/e-ink/how-it-works.md`
- Create: `zh/src/e-ink/circuit.md`
- Create: `zh/src/e-ink/draw-text.md`
- Create: `zh/src/e-ink/draw-image.md`
- Create: `zh/src/e-ink/weather-station/index.md`
- Create: `zh/src/e-ink/weather-station/icons.md`
- Create: `zh/src/e-ink/weather-station/wifi.md`
- Create: `zh/src/e-ink/weather-station/weather-api.md`
- Create: `zh/src/e-ink/weather-station/dashboard.md`
- Create: `zh/src/e-ink/weather-station/main.md`
- Create: `zh/src/tft-display/index.md`
- Create: `zh/src/tft-display/circuit.md`
- Create: `zh/src/tft-display/draw-text.md`
- Create: `zh/src/tft-display/draw-image.md`
- Create: `zh/src/ratatui/index.md`
- Create: `zh/src/ratatui/hello-rust/mousefood.md`
- Create: `zh/src/ratatui/hello-rust/using-mipidsi.md`

- [ ] **Step 1-17: 逐文件翻译**
- [ ] **Step 18: 提交**

```bash
git add zh/src/e-ink/ zh/src/tft-display/ zh/src/ratatui/
git commit -m "feat(zh): translate e-ink, TFT display and ratatui chapters"
```

---

### Task 17: 翻译 Embassy + Projects 收尾

**Files:**
- Create: `zh/src/embassy/index.md`
- Create: `zh/src/embassy/blinky-with-embassy.md`
- Create: `zh/src/embassy/blocking-to-async/index.md`
- Create: `zh/src/projects.md`

- [ ] **Step 1-3: 逐文件翻译**
- [ ] **Step 4: 提交**

```bash
git add zh/src/embassy/ zh/src/projects.md
git commit -m "feat(zh): translate embassy and projects chapters"
```

---

### Task 18: 验证中文版本构建

**Files:**
- Test: `book-zh.toml` 构建结果

- [ ] **Step 1: 构建中文版**

```bash
cd /home/mering/projects/esp32-book
mdbook build --config book-zh.toml
```

Expected: 构建成功，无 ERROR

- [ ] **Step 2: 检查构建输出**

```bash
ls -la book-zh/          # 确认输出目录存在
ls book-zh/index.html    # 确认首页存在
```

- [ ] **Step 3: 检查链接完整性**

```bash
# mdbook 构建时会报告 broken link，检查控制台输出
# 如有 broken link，定位对应 zh/src/ 文件修复
```

- [ ] **Step 4: 抽查渲染效果**

随机打开几个页面检查：
- `book-zh/index.html` — 首页翻译
- `book-zh/std-to-no-std/index.html` — 核心概念
- `book-zh/wifi/index.html` — 外设章节
- 确认图片正常加载、代码块格式正确

- [ ] **Step 5: 提交验证结果**

如有修复，提交修复；如无需修复，此步骤跳过。

---

### Task 19: 最终清理与提交

- [ ] **Step 1: 检查未翻译文件**

```bash
# 对比 src/ 和 zh/src/ 的文件列表
find src -name "*.md" | sed 's|^src|zh/src|' | sort > /tmp/expected.txt
find zh/src -name "*.md" | sort > /tmp/actual.txt
diff /tmp/expected.txt /tmp/actual.txt
# Expected: 无差异（所有文件都已创建）
```

- [ ] **Step 2: 检查是否有 buttons 目录需要处理**

`src/buttons/index.md` 在 SUMMARY.md 中被注释掉了，确认是否需要翻译：
- 如不需要，跳过
- 如需要，补充翻译

- [ ] **Step 3: 最终 git status 检查**

```bash
git status
# 确认所有变更已提交
```

---

## 自我审查

**1. Spec 覆盖率检查：**
- [x] 双语结构（src/ + zh/src/ + book-zh.toml）→ Task 2
- [x] 术语表建立 → Task 1
- [x] SUMMARY.md 翻译 → Task 2
- [x] 正文翻译 → Task 3-17
- [x] 代码注释双语格式 → 贯穿 Task 3-17 的翻译规则
- [x] 术语首次出现标注 → 贯穿 Task 3-17 的翻译规则
- [x] 图片保留原样 → 翻译规则中明确不处理图片
- [x] 构建验证 → Task 18

**2. Placeholder 扫描：**
- [x] 无 "TBD", "TODO", "implement later"
- [x] 无模糊描述
- [x] 每个任务包含具体文件路径
- [x] 每个步骤包含可执行命令

**3. 类型一致性：**
- [x] 文件路径在任务间一致（使用 `zh/src/` 前缀）
- [x] 提交信息格式一致
- [x] 翻译规则在所有任务中统一

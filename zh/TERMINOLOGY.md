# 术语表

## 保留英文（不翻译）

| 术语 | 说明 | 类别 |
|------|------|------|
| no_std | Rust 无标准库特性 | 语言关键字 |
| no_main | Rust 无标准入口点特性 | 语言关键字 |
| std | Rust 标准库 | 语言关键字 |
| core | Rust 核心库（no_std 下可用） | 语言关键字 |
| panic_handler | Rust 恐慌处理属性 | 语言关键字 |
| #![no_std] | Rust 无标准库属性宏 | 语言关键字 |
| #![no_main] | Rust 无标准主函数属性宏 | 语言关键字 |
| #[main] | 自定义入口点属性宏 | 语言关键字 |
| ELF | Executable and Linkable Format | 格式/协议 |
| Cargo | Rust 构建系统和包管理器 | 工具名 |
| cargo-binstall | 用于快速安装 Rust 二进制工具 | 工具名 |
| espflash | Espressif 芯片串行烧录工具 | 工具名 |
| esp-generate | ESP-RS 提供的 no_std 项目模板生成工具 | 工具名 |
| esp-idf-template | ESP-RS 提供的 std 项目模板 | 工具名 |
| espup | ESP 工具链安装工具 | 工具名 |
| rustup | Rust 工具链管理器 | 工具名 |
| rustc | Rust 编译器 | 工具名 |
| probe-rs | 嵌入式调试工具集 | 工具名 |
| svd2rust | 从 SVD 文件生成 PAC 的工具 | 工具名 |
| Embassy | Rust 异步嵌入式框架 | 框架名 |
| esp-hal | ESP32 硬件抽象层 crate | 框架/库名 |
| esp-hal-embassy | esp-hal 与 Embassy 的集成 crate | 框架/库名 |
| esp-rtos | ESP32 实时操作系统（Embassy 运行时） | 框架/库名 |
| esp-alloc | ESP32 堆分配器 crate | 库名 |
| esp-radio | ESP32 Wi-Fi/BLE 控制器初始化 crate | 库名 |
| esp-bootloader-esp-idf | ESP-IDF 引导加载程序支持 crate | 库名 |
| esp-println | ESP32 打印输出 crate | 库名 |
| embedded-hal | 嵌入式硬件抽象层 trait 标准 | 库名 |
| embedded-hal-bus | 嵌入式总线共享库 | 库名 |
| embedded-graphics | 嵌入式 2D 图形库 | 库名 |
| display-interface | 显示接口抽象库 | 库名 |
| display-interface-spi | SPI 显示接口桥接库 | 库名 |
| ssd1306 | SSD1306 OLED 驱动 crate | 库名 |
| ili9341 | ILI9341 TFT 驱动 crate | 库名 |
| mipidsi | MIPI DSI 显示驱动 crate | 库名 |
| mousefood | Ratatui 嵌入式后端（embedded-graphics backend） | 库名 |
| ratatui | Rust 终端用户界面（TUI）库 | 库名 |
| picoserve | 裸机异步 HTTP 服务器 crate | 库名 |
| trouble / TrouBLE | Rust BLE Host 协议栈实现 | 库名 |
| bleps | 轻量级 BLE 外设协议栈 crate | 库名 |
| bt-hci | 蓝牙 HCI 协议 crate | 库名 |
| embassy-time | Embassy 定时器与延时库 | 库名 |
| embassy-net | Embassy 网络支持库 | 库名 |
| embassy-usb | Embassy USB 设备库 | 库名 |
| embassy-futures | Embassy 异步原语库 | 库名 |
| embassy-sync | Embassy 同步原语库 | 库名 |
| embassy-executor | Embassy 异步执行器 | 库名 |
| static_cell | 静态变量运行时初始化库 | 库名 |
| panic_halt | 简单 panic 处理 crate | 库名 |
| defmt | 嵌入式格式化日志框架 | 库名 |
| ESP-RS | Espressif 官方 Rust 生态系统组织 | 组织/项目名 |
| ESP-IDF | Espressif 官方开发框架 | 框架名 |
| HAL | Hardware Abstraction Layer | 硬件抽象层缩写 |
| PAC | Peripheral Access Crate | 外设访问 crate 缩写 |
| BSP | Board Support Package / Board Support Crate | 板级支持包缩写 |
| SVD | System View Description | 系统视图描述文件缩写 |
| MMIO | Memory-Mapped I/O | 内存映射 I/O 缩写 |
| GPIO | General Purpose Input/Output | 硬件缩写 |
| PWM | Pulse Width Modulation | 硬件/协议缩写 |
| LEDC | LED PWM Controller（ESP32 外设） | 硬件缩写 |
| MCPWM | Motor Control Pulse Width Modulator（ESP32 外设） | 硬件缩写 |
| ADC | Analog-to-Digital Converter | 硬件/协议缩写 |
| SAR ADC | Successive Approximation Register ADC | 硬件/协议缩写 |
| I2C | Inter-Integrated Circuit | 硬件/协议缩写 |
| SPI | Serial Peripheral Interface | 硬件/协议缩写 |
| UART | Universal Asynchronous Receiver/Transmitter | 硬件/协议缩写 |
| SCL | Serial Clock Line（I2C） | 硬件/协议缩写 |
| SDA | Serial Data Line（I2C） | 硬件/协议缩写 |
| SCK | Serial Clock（SPI） | 硬件/协议缩写 |
| MOSI | Master Out, Slave In（SPI） | 硬件/协议缩写 |
| MISO | Master In, Slave Out（SPI） | 硬件/协议缩写 |
| CS | Chip Select（SPI） | 硬件/协议缩写 |
| SDO | Serial Data Out | 硬件/协议缩写 |
| SDI | Serial Data In | 硬件/协议缩写 |
| SS | Slave Select（SPI 旧称） | 硬件/协议缩写 |
| DMA | Direct Memory Access | 硬件/协议缩写 |
| RTC | Real-Time Clock | 硬件/协议缩写 |
| TIMG | Timer Group（ESP32 外设） | 硬件缩写 |
| BLE | Bluetooth Low Energy | 硬件/协议缩写 |
| BR/EDR | Basic Rate/Enhanced Data Rate（蓝牙经典模式） | 硬件/协议缩写 |
| GAP | Generic Access Profile（BLE） | 硬件/协议缩写 |
| GATT | Generic Attribute Profile（BLE） | 硬件/协议缩写 |
| ATT | Attribute Protocol（BLE） | 硬件/协议缩写 |
| HCI | Host Controller Interface（BLE） | 硬件/协议缩写 |
| L2CAP | Logical Link Control and Adaptation Protocol | 硬件/协议缩写 |
| UUID | Universally Unique Identifier | 通用缩写 |
| UID | Unique Identifier | 通用缩写 |
| Wi-Fi | Wireless Fidelity | 硬件/协议缩写 |
| AP | Access Point（Wi-Fi 接入点模式） | 硬件/协议缩写 |
| STA | Station（Wi-Fi 站点模式） | 硬件/协议缩写 |
| DHCP | Dynamic Host Configuration Protocol | 网络协议缩写 |
| HTTP | HyperText Transfer Protocol | 网络协议缩写 |
| TCP | Transmission Control Protocol | 网络协议缩写 |
| IP | Internet Protocol | 网络协议缩写 |
| SSID | Service Set Identifier（Wi-Fi 网络名称） | 网络协议缩写 |
| RFID | Radio Frequency Identification | 硬件/协议缩写 |
| PCD | Proximity Coupling Device（RFID 读卡器） | 硬件/协议缩写 |
| PICC | Proximity Integrated Circuit Card（RFID 标签） | 硬件/协议缩写 |
| EEPROM | Electrically Erasable Programmable Read-Only Memory | 硬件缩写 |
| FIFO | First In, First Out（缓冲区） | 通用缩写 |
| FAT32 | File Allocation Table 32 | 文件系统缩写 |
| SD | Secure Digital（SD 卡） | 硬件缩写 |
| SDC | Secure Digital Card | 硬件缩写 |
| MMC | MultiMediaCard | 硬件缩写 |
| SoC | System on Chip | 硬件缩写 |
| MCU | Microcontroller Unit | 硬件缩写 |
| CPU | Central Processing Unit | 硬件缩写 |
| IC | Integrated Circuit | 硬件缩写 |
| PCB | Printed Circuit Board | 硬件缩写 |
| EMI | Electromagnetic Interference | 硬件缩写 |
| FCC | Federal Communications Commission | 认证机构缩写 |
| CE | Conformite Europeenne（欧盟合格认证） | 认证标识 |
| LDR | Light Dependent Resistor | 硬件缩写 |
| NTC | Negative Temperature Coefficient（热敏电阻） | 硬件缩写 |
| PTC | Positive Temperature Coefficient（热敏电阻） | 硬件缩写 |
| PIR | Passive Infrared（被动红外传感器） | 硬件缩写 |
| LCD | Liquid Crystal Display | 硬件缩写 |
| OLED | Organic Light-Emitting Diode | 硬件缩写 |
| TFT | Thin-Film Transistor | 硬件缩写 |
| e-ink / e-Paper | electronic ink / electronic paper | 硬件名 |
| GDDRAM | Graphics Display DRAM（OLED 显存） | 硬件缩写 |
| ESP32 | Espressif 32-bit 系列微控制器 | 硬件型号 |
| ESP32-S3 | ESP32 S 系列芯片（带 AI 加速） | 硬件型号 |
| ESP32-C3 | ESP32 C 系列芯片（RISC-V） | 硬件型号 |
| ESP32-C6 | ESP32 C 系列芯片（Wi-Fi 6） | 硬件型号 |
| ESP32-H2 | ESP32 H 系列芯片（低功耗 BLE） | 硬件型号 |
| ESP32-P4 | ESP32 P 系列芯片（应用处理器） | 硬件型号 |
| ESP32-S2 | ESP32 S 系列芯片（无 BLE） | 硬件型号 |
| ESP32-C2 | ESP32 C 系列芯片（低成本） | 硬件型号 |
| ESP32-C5 | ESP32 C 系列芯片 | 硬件型号 |
| WROOM | Espressif 预认证 Wi-Fi/蓝牙模块系列 | 硬件型号 |
| WROVER | Espressif 带 PSRAM 的模块系列 | 硬件型号 |
| DevKit / Devkit | Development Kit（开发板） | 硬件型号 |
| DevKit V1 | ESP32 开发板型号（本书使用） | 硬件型号 |
| ESP32-S3-DevKitC | ESP32-S3 开发板型号 | 硬件型号 |
| SG90 | 微型舵机型号 | 硬件型号 |
| HC-SR04 | 超声波测距传感器型号 | 硬件型号 |
| HC-SR501 | 被动红外人体感应传感器型号 | 硬件型号 |
| RC522 | RFID 读卡器模块型号（基于 MFRC522） | 硬件型号 |
| MFRC522 | NXP 设计的 RFID 读卡器 IC | 硬件型号 |
| MIFARE | NXP 非接触式智能卡产品系列 | 硬件/协议名 |
| MIFARE Classic | MIFARE 经典系列智能卡 | 硬件/协议名 |
| MIFARE Classic EV1 | MIFARE Classic EV1 1K/4K 智能卡 | 硬件/协议名 |
| SSD1306 | OLED 显示控制器芯片 | 硬件型号 |
| HD44780 | 字符 LCD 控制器芯片（日立） | 硬件型号 |
| ILI9341 | TFT 显示控制器芯片 | 硬件型号 |
| XPT2046 | 电阻式触摸屏控制器芯片 | 硬件型号 |
| BISS0001 | PIR 传感器模块控制器芯片 | 硬件型号 |
| Waveshare | 电子模块品牌（微雪） | 品牌名 |
| Xtensa | Tensilica 处理器架构（ESP32 使用） | 架构名 |
| RISC-V | 开源指令集架构 | 架构名 |
| ARM Cortex | ARM 处理器核心架构系列 | 架构名 |
| Rust | Rust 编程语言 | 语言名 |
| async / await | Rust 异步编程关键字 | 语言关键字 |
| trait | Rust 特性/特征 | 语言关键字 |
| crate | Rust 包/库单元 | 语言关键字 |
| feature flag | Rust 特性标志（Cargo.toml 中启用功能） | 语言/工具概念 |
| build.rs | Rust 构建脚本文件名 | 文件名 |
| .cargo/config.toml | Cargo 配置文件路径 | 文件名 |
| rust-toolchain.toml | Rust 工具链配置文件 | 文件名 |
| Cargo.toml | Rust 项目依赖与元数据配置文件 | 文件名 |
| lib.rs | Rust 库入口文件名 | 文件名 |
| main.rs | Rust 可执行程序入口文件名 | 文件名 |
| target triple | 目标三元组（架构-厂商-系统-ABI） | 编译概念 |
| linker script | 链接器脚本（文件名为 linkall.x 等） | 编译概念 |
| prescaler | 预分频器（时钟分频） | 硬件/编译概念 |
| nRF Connect | Nordic 提供的 BLE 调试移动应用 | 工具/应用名 |
| Musescore | 乐谱分享网站/软件 | 工具/网站名 |
| Falstad | 电路仿真网站 | 工具/网站名 |

---

## 翻译并标注英文（首次出现）

| 英文 | 中文 | 备注 |
|------|------|------|
| peripheral | 外设 | 通用 |
| toolchain | 工具链 | 通用 |
| cross compilation | 交叉编译 | 通用 |
| bare metal | 裸机 | 通用 |
| linker script | 链接器脚本 | 通用 |
| flash (v.) | 烧录 | 动词，视上下文也可用"刷写" |
| firmware | 固件 | 通用 |
| interrupt | 中断 | 通用 |
| register | 寄存器 | 通用 |
| datasheet | 数据手册 | 通用 |
| breadboard | 面包板 | 通用 |
| pinout | 引脚图 | 通用 |
| jumper wire | 跳线 | 通用 |
| resistor | 电阻 | 通用 |
| potentiometer | 电位器 | 通用 |
| voltage divider | 分压器 / 电压分压电路 | 通用 |
| anode | 阳极（正极） | 电子元件 |
| cathode | 阴极（负极） | 电子元件 |
| onboard LED | 板载 LED | 通用 |
| external LED | 外接 LED | 通用 |
| drive mode | 驱动模式 | 通用，如 PushPull、OpenDrain |
| push-pull | 推挽输出 | 驱动模式 |
| open drain | 开漏输出 | 驱动模式 |
| duty cycle | 占空比 | PWM 相关 |
| frequency | 频率 | 通用 |
| period | 周期 | 通用 |
| pulse width | 脉冲宽度 | PWM/伺服相关 |
| resolution | 分辨率 | ADC/PWM 相关 |
| clock source | 时钟源 | 通用 |
| prescaler | 预分频器 | 时钟相关 |
| watchdog timer | 看门狗定时器 | 通用 |
| delay | 延时 / 延迟 | 通用 |
| blocking delay | 阻塞延时 | 与 async 非阻塞相对 |
| singleton pattern | 单例模式 | 设计模式 |
| offloading | 卸载 / 任务卸载 | 将任务从 CPU 卸载到外设 |
| embedded systems | 嵌入式系统 | 通用 |
| microcontroller | 微控制器 | 通用 |
| development board | 开发板 | 通用 |
| module | 模块 | 硬件模块 |
| RF module | 射频模块 | 通用 |
| antenna | 天线 | 通用 |
| crystal oscillator | 晶振 | 通用 |
| voltage regulator | 稳压器 | 通用 |
| level shifter | 电平转换器 | 通用 |
| logic level | 逻辑电平 | 通用 |
| pull-up resistor | 上拉电阻 | 通用 |
| pull-down resistor | 下拉电阻 | 通用 |
| floating pin | 浮空引脚 | 通用 |
| input-only pin | 仅输入引脚 | ESP32 GPIO 类型 |
| GPIO tolerance | GPIO 耐压值 | 通用 |
| ground (GND) | 地线 / 接地 | 通用 |
| power supply | 电源 | 通用 |
| VCC | 正电源电压 | 通用 |
| Vout | 输出电压 | 通用 |
| Vin | 输入电压 | 通用 |
| Vref | 参考电压 | ADC 相关 |
| baud rate | 波特率 | 串口通信 |
| acknowledge (ACK) | 应答 / 确认 | 通信协议 |
| start condition | 起始条件 | I2C 通信 |
| stop condition | 停止条件 | I2C 通信 |
| full duplex | 全双工 | 通信模式 |
| half duplex | 半双工 | 通信模式 |
| synchronous | 同步 | 通信模式 |
| serial communication | 串行通信 | 通用 |
| master-slave architecture | 主从架构 | 通信架构（现多称 controller-target） |
| controller | 控制器 / 主设备 | I2C/SPI 通信中的主控端 |
| target | 目标设备 / 从设备 | I2C 通信中的受控端（原 slave） |
| chip select | 片选 | SPI 通信 |
| address | 地址 | I2C 设备地址等 |
| speed mode | 速度模式 | I2C 标准模式/快速模式等 |
| SPI mode | SPI 模式（0~3） | SPI 时钟极性与相位配置 |
| idle level | 空闲电平 | SPI 模式相关 |
| edge | 边沿（上升沿/下降沿） | 数字信号 |
| active low | 低电平有效 | 数字信号 |
| active high | 高电平有效 | 数字信号 |
| analog signal | 模拟信号 | 通用 |
| digital signal | 数字信号 | 通用 |
| sensor | 传感器 | 通用 |
| actuator | 执行器 | 通用 |
| servo motor | 舵机 / 伺服电机 | 通用 |
| horn | 舵盘 / 舵机摇臂 | 伺服电机输出轴连接件 |
| gearbox | 减速齿轮箱 | 伺服电机内部 |
| DC motor | 直流电机 | 通用 |
| buzzer | 蜂鸣器 | 通用 |
| active buzzer | 有源蜂鸣器 | 内置振荡器 |
| passive buzzer | 无源蜂鸣器 | 需外部 PWM 驱动 |
| thermistor | 热敏电阻 | 通用 |
| LDR (Light Dependent Resistor) | 光敏电阻 | 通用 |
| photocell | 光电池 / 光敏电阻 | 与 LDR 同义 |
| photoresistor | 光敏电阻 | 与 LDR 同义 |
| ultrasonic sensor | 超声波传感器 | 通用 |
| transmitter | 发射器 | 通用 |
| receiver | 接收器 | 通用 |
| transducer | 换能器 | 通用 |
| trigger pin | 触发引脚 | HC-SR04 |
| echo pin | 回响引脚 | HC-SR04 |
| PIR sensor | 被动红外传感器 / 人体红外传感器 | 通用 |
| pyroelectric sensor | 热释电传感器 | PIR 核心元件 |
| Fresnel lens | 菲涅尔透镜 | PIR 传感器外罩 |
| joystick | 摇杆 / 操纵杆 | 通用 |
| knob | 旋钮 / 摇杆帽 | 通用 |
| RFID tag | RFID 标签 | 通用 |
| RFID reader | RFID 读卡器 | 通用 |
| key fob | 钥匙扣标签 | RFID 卡片形式 |
| smart card | 智能卡 | 通用 |
| contactless | 非接触式 | 通用 |
| sector | 扇区 | MIFARE 存储结构 |
| block | 块 | MIFARE 存储结构 |
| trailer block | 尾部块 / 尾块 | MIFARE 每个扇区的最后一块 |
| access bits | 访问位 | MIFARE 权限控制 |
| authentication key | 认证密钥 | MIFARE KeyA/KeyB |
| read/write block | 读写块 | MIFARE 数据块类型 |
| value block | 数值块 | MIFARE 数据块类型 |
| LCD display | LCD 显示屏 | 通用 |
| backlight | 背光 | 显示相关 |
| contrast | 对比度 | 显示相关 |
| pixel | 像素 | 显示相关 |
| resolution | 分辨率 | 显示相关 |
| segment | 段 / 列（OLED） | OLED 显示结构 |
| common | 公共端 / 行（OLED） | OLED 显示结构 |
| page (display) | 页（OLED 显存） | OLED GDDRAM 结构 |
| custom character | 自定义字符 | LCD 相关 |
| ASCII character | ASCII 字符 | 通用 |
| OLED display | OLED 显示屏 | 通用 |
| monochrome | 单色 | 显示相关 |
| TFT display | TFT 显示屏 | 通用 |
| touchscreen | 触摸屏 | 通用 |
| resistive touchscreen | 电阻式触摸屏 | 通用 |
| display driver | 显示驱动 / 显示驱动芯片 | 通用 |
| controller chip | 控制芯片 | 通用 |
| e-ink display | 电子墨水屏 / 电纸屏 | 通用 |
| refresh mode | 刷新模式 | 电子墨水屏相关 |
| full refresh | 全局刷新 | 电子墨水屏 |
| partial refresh | 局部刷新 | 电子墨水屏 |
| ghosting | 残影 | 显示相关 |
| SD card | SD 卡 | 通用 |
| microSD card | 微型 SD 卡 | 通用 |
| SD card adapter | SD 卡适配器 / 读卡器模块 | 通用 |
| block device | 块设备 | 存储相关 |
| heap | 堆 | 内存相关 |
| stack | 栈 | 内存相关 |
| RAM | 随机存取存储器 | 硬件缩写（保留英文） |
| SRAM | 静态随机存取存储器 | 硬件缩写（保留英文） |
| ROM | 只读存储器 | 硬件缩写（保留英文） |
| Flash memory | 闪存 | 通用 |
| PSRAM | 伪静态随机存取存储器 | 硬件缩写（保留英文） |
| bootloader | 引导加载程序 | 通用 |
| reset handler | 复位处理程序 | 通用 |
| app descriptor | 应用描述符 | ESP-IDF 引导相关 |
| radio | 射频 / 无线电 | Wi-Fi/BLE 共享射频 |
| network stack | 网络协议栈 | 通用 |
| socket | 套接字 | 网络相关 |
| router | 路由器 | 网络相关 |
| static IP | 静态 IP | 网络相关 |
| DHCP | 动态主机配置协议 | 保留英文缩写 |
| web server | Web 服务器 | 通用 |
| routing | 路由 | Web/网络相关 |
| endpoint | 端点 | API/Web 相关 |
| request | 请求 | HTTP 相关 |
| response | 响应 | HTTP 相关 |
| advertising | 广播 / 广告（BLE） | BLE GAP |
| advertisement | 广播数据包（BLE） | BLE GAP |
| discoverable | 可发现 | BLE GAP |
| connection-oriented | 面向连接 | BLE 通信类型 |
| connection-less | 无连接 | BLE 通信类型 |
| central | 中心设备（BLE） | GAP 角色，如手机 |
| peripheral | 外设（BLE） | GAP 角色，如传感器 |
| broadcaster | 广播者（BLE） | GAP 角色 |
| observer | 观察者（BLE） | GAP 角色 |
| server (GATT) | 服务器（GATT） | GATT 角色 |
| client (GATT) | 客户端（GATT） | GATT 角色 |
| service (GATT) | 服务（GATT） | GATT 数据结构 |
| characteristic (GATT) | 特征（GATT） | GATT 数据结构 |
| profile (GATT) | 配置文件（GATT） | GATT 数据结构 |
| attribute | 属性 | BLE ATT 协议 |
| handle | 句柄 | BLE ATT 属性标识 |
| permission | 权限 | BLE ATT 属性权限 |
| notify | 通知（BLE） | GATT 特征属性 |
| indicate | 指示（BLE） | GATT 特征属性 |
| subscribe | 订阅（BLE） | GATT 客户端行为 |
| task | 任务 | 异步/RTOS 相关 |
| spawner | 任务生成器 | Embassy 概念 |
| executor | 执行器 | 异步运行时 |
| runtime | 运行时 | 通用 |
| non-blocking | 非阻塞 | 异步编程 |
| blocking | 阻塞 | 异步编程 |
| await | 等待（async await） | 异步编程 |
| future | 未来 / Future | Rust 异步类型 |
| pool size | 任务池大小 | Embassy 任务属性 |
| timer | 定时器 | 通用 |
| timeout | 超时 | 通用 |
| tick | 时钟节拍 / 滴答 | 定时器相关 |
| timestamp | 时间戳 | 通用 |
| prescaler | 预分频器 | 定时器/时钟相关 |
| compare value | 比较值 | PWM 定时器 |
| overflow interrupt | 溢出中断 | 定时器相关 |
| compare interrupt | 比较中断 | 定时器相关 |
| dead time | 死区时间 | MCPWM 电机控制 |
| fault handler | 故障处理程序 | MCPWM |
| capture module | 捕获模块 | MCPWM |
| Space Vector PWM (SVPWM) | 空间矢量脉宽调制 | 电机控制 |
| Field Oriented Control (FOC) | 磁场定向控制 | 电机控制 |
| brushed motor | 有刷电机 | 电机类型 |
| brushless motor | 无刷电机 | 电机类型 |
| RC servo motor | 航模舵机 | 通用 |
| power DAC | 电源数模转换器 | MCPWM 应用 |
| duty resolution | 占空比分辨率 | PWM 相关 |
| clock divider | 时钟分频器 | PWM/定时器相关 |
| high-speed channel | 高速通道 | ESP32 LEDC |
| low-speed channel | 低速通道 | ESP32 LEDC |
| glitch-free | 无 glitch / 无毛刺切换 | PWM 相关 |
| fade | 渐变 / 淡入淡出 | LED 亮度渐变 |
| duty fade | 占空比渐变 | LEDC 功能 |
| song / melody | 歌曲 / 旋律 | 蜂鸣器相关 |
| note | 音符 | 音乐理论 |
| octave | 八度 | 音乐理论 |
| tempo | 速度 / 拍速 | 音乐理论 |
| BPM (Beats Per Minute) | 每分钟节拍数 | 音乐理论（保留英文缩写） |
| whole note | 全音符 | 音乐理论 |
| half note | 二分音符 | 音乐理论 |
| quarter note | 四分音符 | 音乐理论 |
| eighth note | 八分音符 | 音乐理论 |
| sixteenth note | 十六分音符 | 音乐理论 |
| dotted note | 附点音符 | 音乐理论 |
| beat | 拍子 | 音乐理论 |
| weather station | 气象站 | 项目名 |
| dashboard | 仪表盘 / 控制面板 | 通用 |
| TUI (Terminal User Interface) | 终端用户界面 | 通用（保留英文缩写） |
| widget | 控件 / 小部件 | UI 相关 |
| backend | 后端 / 后台 | Ratatui 后端 |
| layout | 布局 | UI 相关 |
| frame | 帧 / 画面 | UI/Ratatui 相关 |
| render | 渲染 | 图形/UI |
| project template | 项目模板 | 通用 |
| boilerplate code | 样板代码 | 通用 |
| feature flag | 特性标志 | Rust/Cargo |
| unstable feature | 不稳定特性 | Rust |
| nightly | Nightly 工具链 | Rust 发布通道 |
| allocation | 堆分配 / 内存分配 | 通用 |
| heap allocator | 堆分配器 | 内存相关 |
| static variable | 静态变量 | 通用 |
| macro | 宏 | Rust |
| trait implementation | trait 实现 | Rust |
| type alias | 类型别名 | Rust |
| enum | 枚举 | Rust |
| struct | 结构体 | Rust |
| module | 模块 | Rust |
| submodule | 子模块 | Rust |
| lib.rs | 库根文件 | Rust 文件名（保留英文） |
| bin | 二进制目标 | Rust |
| workspace | 工作区 | Cargo |
| dependency | 依赖 | 通用 |
| version | 版本 | 通用 |
| repository | 仓库 / 代码库 | 通用 |
| commit | 提交 | Git |
| pull request (PR) | 拉取请求 | Git |
| issue | 问题 / Issue | GitHub |
| license | 许可证 | 通用 |
| MIT License | MIT 许可证 | 保留英文 |
| Apache License | Apache 许可证 | 保留英文 |
| CC-BY-SA | 署名-相同方式共享（知识共享许可协议） | 保留英文缩写 |
| IoT (Internet of Things) | 物联网 | 通用（保留英文缩写） |
| hands-on | 动手实践 | 通用 |

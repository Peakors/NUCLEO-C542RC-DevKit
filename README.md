# NUCLEO-C542RC-DevKit

基于 ST **NUCLEO-C542RC** 开发板的文档与工程模板仓库。开发主线：**STM32CubeMX2 + CLion + ST-LINK**（板载 STLINK-V3EC），宿主系统 macOS（Apple Silicon）。

## 硬件速查

| 项目 | 参数 |
| --- | --- |
| MCU | STM32C542RCT6，Arm Cortex-M33，最高 144 MHz，LQFP64 |
| 存储 | 256 KB Flash / 64 KB RAM |
| 板载调试器 | STLINK-V3EC（USB Type-C，支持 VCP、SWD、虚拟 U 盘） |
| 用户 LED | LD1 → **PA5**，高电平点亮（Arduino D13） |
| 用户按键 | B1 → **PC13**（下拉，按下为高）；B2 复位；B3 进入 BOOT |
| 虚拟串口 | USART2（**PA2/PA3**）经 SB12/SB26 连到 STLINK-V3EC |
| 晶振 | HSE 24 MHz、LSE 32.768 kHz |
| 扩展接口 | Arduino Uno V3、ST morpho、MIPI10 调试口、CAN FD |

> 依据 UM3615（2026-02）与 ST 数据手册；详见 [docs/01-硬件速览.md](docs/01-硬件速览.md)。

## 文档

| 文档 | 内容 |
| --- | --- |
| [docs/01-硬件速览.md](docs/01-硬件速览.md) | 板卡资源、关键引脚、电源与启动模式 |
| [docs/02-环境搭建.md](docs/02-环境搭建.md) | macOS 上安装 CubeMX2、CubeCLT、CLion，校验 ST-LINK |
| [docs/03-CubeMX2创建工程.md](docs/03-CubeMX2创建工程.md) | 用 CubeMX2 建 NUCLEO-C542RC 工程、点灯、生成 CMake |
| [docs/04-CLion导入编译调试.md](docs/04-CLion导入编译调试.md) | CLion 打开工程、编译、ST-LINK 断点调试、外设寄存器 |
| [docs/05-下载与调试.md](docs/05-下载与调试.md) | CLion 内置 ST-LINK 下载/调试、串口、CLI 兜底 |
| [docs/06-常见问题.md](docs/06-常见问题.md) | 连接、串口、生成器、CMake/pack 等高频问题 |

## 模板

| 模板 | 用途 |
| --- | --- |
| [templates/clion/README.md](templates/clion/README.md) | CLion 中 ST-LINK GDB server 调试配置参数 |
| [templates/project-template/README.md](templates/project-template/README.md) | 新建工程时复制使用的检查清单 |

## 仓库结构

```text
.
├── AGENTS.md          # AI 助手与协作者约定
├── README.md
├── docs/              # 中文文档
├── templates/         # 配置模板
├── projects/          # 实际工程（待创建）
└── .gitignore
```

## 快速开始

1. 按 [docs/02-环境搭建.md](docs/02-环境搭建.md) 准备工具链。
2. 按 [docs/03-CubeMX2创建工程.md](docs/03-CubeMX2创建工程.md) 用 CubeMX2 生成 CMake 工程。
3. 按 [docs/04-CLion导入编译调试.md](docs/04-CLion导入编译调试.md) 在 CLion 里编译并单步调试。

## 状态

- ✅ 仓库、文档骨架、模板已就绪
- ⏳ 待补充：CubeMX2 实操截图、`projects/` 示例工程、printf 重定向示例
- ✅ 下载/调试统一使用 CLion 内置 ST-LINK（不使用 OpenOCD）

## 许可证

[MIT License](LICENSE)

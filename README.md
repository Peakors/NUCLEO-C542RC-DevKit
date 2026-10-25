# NUCLEO-C542RC-DevKit

基于 ST **NUCLEO-C542RC**（STM32C542RCT6）开发板的个人工程仓库。

> 当前状态：仓库初始化完成，固件工程（STM32CubeMX 工程 / CMake + arm-none-eabi 构建）后续提交。

## 目录结构

```text
.
├── README.md          # 项目说明
├── LICENSE            # MIT 许可证
├── .gitignore         # 构建产物与 IDE 本地文件忽略规则
└── .gitattributes     # 换行符与二进制文件规则
```

后续计划加入：

| 目录 / 文件 | 说明 |
| --- | --- |
| `Core/` | CubeMX 生成的应用代码、中断向量与启动文件 |
| `Drivers/` | CMSIS 与 STM32 HAL 驱动 |
| `docs/` | 开发笔记、原理图与调试记录 |
| `CMakeLists.txt` | arm-none-eabi-gcc 交叉编译入口 |
| `*.ioc` | STM32CubeMX 工程配置 |

## 开发环境

本机参考环境（按需调整）：

- **STM32CubeCLT 1.22.0** — 自带 arm-none-eabi-gcc、CMake、Ninja、OpenOCD
- **STM32CubeMX** — 生成初始化代码与 `.ioc` 工程
- **ST-LINK** 调试器与 OpenOCD（`st-link` 接口）

## 构建与烧录

工程代码加入后补充构建、下载命令。

## 许可证

本项目采用 [MIT License](LICENSE)。

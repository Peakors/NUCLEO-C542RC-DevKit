# 04 · CLion 导入、编译与调试

> 适用版本：CLion 2026.1 / 2026.2 · STM32CubeCLT 1.22.0 · CubeMX2 1.1.2 生成的 CMake 工程
> 前置条件：已完成 [03-CubeMX2创建工程](03-CubeMX2创建工程.md)，工程能通过 `cmake --build` 编译
> 验证方式：CLion 中能编译、能下载、能在 `main()` 打断点并单步执行，外设寄存器可见

> 调试与下载使用 **CLion 内置的 ST-LINK 支持**，不需要 OpenOCD，也不需要手写 `.cfg` 脚本。

## 1. 打开工程

1. `File > Open`，选择 CubeMX2 生成的**工程目录**（包含 `CMakeLists.txt` 的那一层）。
2. CLion 会按 CMake 工程加载，右下角出现 CMake 配置进度。
3. 首次打开若提示 “CMake project is not loaded”，点 **Load CMake Project**。

> 不要只打开单个 `.c` 文件；CLion 需要 `CMakeLists.txt` 才能建立索引。

## 2. 配置工具链

`Settings > Build, Execution, Deployment > Toolchains`：

- 选择 CubeCLT 工具链（若没有，按 [02-环境搭建](02-环境搭建.md) 第 4 节手动添加）。
- 确认 C/C++ 编译器、调试器、CMake、Ninja 均指向 `/opt/ST/STM32CubeCLT_1.22.0/...`，路径后面没有红色报错。

`Settings > Build, Execution, Deployment > CMake`：

| 字段 | 建议值 |
| --- | --- |
| Name | `Debug` |
| Build type | `Debug` |
| Generator | `Ninja` |
| CMake options | 留空（CubeMX2 已生成所需定义） |
| Build directory | `build/debug`（与命令行构建目录区分） |
| Toolchain | 上一步的 CubeCLT 工具链 |

修改后点 **Apply**，等待 CMake reload 完成。

## 3. 编译

- 快捷键 `Cmd + F9`（Build Project），或点击工具栏锤子图标。
- 产物路径：`build/debug/<PROJECT>.elf`（以 CMake 配置为准）。

预期：Build 面板输出 `[100%] Built target ...`，无 error。

> 若报 `CMake 3.30 or higher is required`，说明 CLion 用到了系统旧版 CMake——把 CMake profile 的 CMake 路径显式指向 `<CubeCLT>/CMake/bin/cmake`。

## 4. 配置 CLion 内置 ST-LINK

CLion 自带 **ST-LINK** 调试模板，内部调用 ST 官方 `ST-LINK_gdbserver`，与 OpenOCD 无关。

| CLion 版本 | 菜单路径 |
| --- | --- |
| 2026.2+ | `Settings > Build, Execution, Deployment > Debugger > Debug Profiles` → **ST-LINK** |
| 2026.1 | `Settings > Build, Execution, Deployment > Debugger > Debug Servers` → `+` → **ST-LINK** |

2026.1 若看不到该页面：`Settings > Advanced Settings > Debugger` 勾选 **Enable Debug Servers**。

按下表填写：

| 字段 | 值 |
| --- | --- |
| Name | `NUCLEO-C542RC ST-LINK` |
| ST-LINK GDB server | `/opt/ST/STM32CubeCLT_1.22.0/STLink-gdb-server/bin/ST-LINK_gdbserver` |
| Programmer path | `/opt/ST/STM32CubeCLT_1.22.0/STM32CubeProgrammer/bin` |
| Interface | **SWD** |
| Device | `STM32C542RCT6` |
| Debugger | `/opt/ST/STM32CubeCLT_1.22.0/GNU-tools-for-STM32/bin/arm-none-eabi-gdb` |
| Executable | `build/debug/<PROJECT>.elf` |
| Download | **Always** |
| Port | `61234` |

配置完成后，该 profile 会出现在主工具栏的配置下拉框里。

> ⚠️ 未实测：CLion 各小版本的 ST-LINK 模板字段名略有差异；关键是 GDB server 路径、Programmer 路径（`-cp`）、SWD、可执行文件、下载策略五项。
> 若模板默认指向 STM32CubeIDE 内的 GDB server，把路径改为上面的 CubeCLT 版本即可。

## 5. 开始调试

1. 在 `main()` 的 `while (1)` 里打一个断点。
2. 点击 **Debug**（`Ctrl + D`）。
3. 预期：
   - Console 出现 `Debugger connected to ST-LINK`；
   - 程序停在断点；
   - 继续运行后 LD1 按代码节奏闪烁。

调试面板可用：

- **Variables / Watches**：查看变量、外设结构体。
- **Live Watches**：ST-LINK 支持实时监视全局变量。
- **Registers**：内核寄存器。
- **Peripherals**：需要 SVD 文件（见下节）。
- **Memory**：查看 Flash/RAM 地址。

## 6. 外设寄存器视图（SVD）

CLion 会从 CMSIS pack 或手动指定的 SVD 读取外设定义。CubeCLT 自带 `STM32C542.svd`：

```text
<CubeCLT>/STMicroelectronics_CMSIS_SVD/STM32C542.svd
```

在 `Settings > Build, Execution, Deployment > Embedded Development` 中检查 SVD 路径；若列表为空，手动添加上述文件，然后重新进入调试即可在 **Peripherals** 页看到 GPIOA、USART2 等寄存器。

## 7. 常用技巧

- **只下载不调试**：用同一套 ST-LINK 配置以 Run 方式启动；或命令行 `STM32_Programmer_CLI`（见 [05](05-下载与调试.md)）。
- **复位后停在 main**：在 ST-LINK 配置里选择 reset 策略；或排查时手动给 GDB server 加 `-k`。
- **串口输出**：调试时 VCP 仍可用，另开终端 `screen /dev/cu.usbmodemXXXX 115200`。
- **多工程共存**：每个工程独立 `build/` 目录，避免 CMake cache 互相污染。
- **探针被占用**：同一时间只允许一个程序使用 ST-LINK（关闭 CubeProgrammer 图形界面、其他调试会话）。

## 8. 下一步

- 下载 / 调试的完整说明（含 CLI 兜底）：[05-下载与调试](05-下载与调试.md)
- 连不上板子：[06-常见问题](06-常见问题.md)

## 参考资料

- CLion STM32CubeMX 工程：<https://www.jetbrains.com/help/clion/embedded-stm32.html>
- CLion Debug servers（ST-LINK 模板）：<https://www.jetbrains.com/help/clion/debug-servers.html>

# AGENTS.md

本文件写给在本仓库里工作的 AI 编码助手（DeepSeek Harness、Codex、Claude Code 等），以及人类协作者。
**动手前请通读；当它与你平时的默认习惯冲突时，以本文件为准。**

---

## 1. 仓库定位

- 主题：**NUCLEO-C542RC** 开发板（MCU：STM32C542RCT6，Arm Cortex-M33，最高 144 MHz）。
- 形态：**学习 / 示例 / 模板仓库**，由「文档 + 可复制模板」组成，不是量产固件仓库。
- 主线工具链：**STM32CubeMX2 → CMake 工程 → CLion → ST-LINK**（板载 STLINK-V3EC），宿主系统为 macOS（Apple Silicon）。
- 当前阶段：文档与模板建设中。`projects/` 目录下目前没有实际工程，新增工程请遵循第 4 节约定。

## 2. 硬性规则

1. **不臆造硬件事实。** 引脚、封装、时钟、外设归属必须来自仓库内文档引用的官方资料（UM3615、Data brief、原理图 MB2213）或本机实测。查不到就写 `⚠️ 待核实`，不要猜。
2. **不手改生成代码。** CubeMX2 生成的 `Core/`、`Drivers/`、CMake 生成文件等，只能通过修改 `.ioc2` 后重新生成来变更；业务代码写在 `USER CODE` 区或独立文件中（见第 6 节）。
3. **不提交构建产物与二进制。** `build/`、`*.elf`、`*.bin`、`*.hex`、`.DS_Store`、packs 缓存（`misc/`）一律不进仓库。
4. **不擅自安装系统级软件、不改全局配置。** 不执行 `sudo`、不修改 `~/.gitconfig`、不安装 brew 包，除非用户明确要求。
5. **不执行破坏性 Git 操作。** 禁止 `git push --force`、`git reset --hard` 到远端已有提交、改写历史、删除远端分支，除非用户逐条确认。
6. **文档必须可复现。** 每个操作步骤都要写清「界面路径 / 命令 / 预期结果」；未实测的内容用 `> ⚠️ 未实测：<依据>` 标注。
7. **改动后自检。** 新增/修改文档后，至少检查链接、命令路径、版本号是否与第 3 节一致。

## 3. 环境事实（2026-10 本机实测，改动机器请同步更新）

| 组件 | 版本 | 路径 / 说明 |
| --- | --- | --- |
| macOS | 26.x（Apple Silicon） | 开发宿主机 |
| STM32CubeMX2 | 文档按 **1.1.2** 编写 | 独立安装；官方文档 <https://dev.st.com/stm32cube-docs/stm32cubemx2/1.1.2/en/index.html> |
| STM32CubeMX（经典版） | 6.18 | `/Applications/STM32Cube/STM32CubeMX.app`，**不用于本文档主流程** |
| STM32CubeCLT | 1.22.0 | `/opt/ST/STM32CubeCLT_1.22.0/` |
| arm-none-eabi-gcc | 14.3.1 | `/opt/ST/STM32CubeCLT_1.22.0/GNU-tools-for-STM32/bin/` |
| CMake | 4.3.1 | `/opt/ST/STM32CubeCLT_1.22.0/CMake/bin/cmake`（CubeMX2 要求 ≥ 3.30） |
| Ninja | 随 CubeCLT | `/opt/ST/STM32CubeCLT_1.22.0/Ninja/bin/ninja` |
| ST-LINK GDB server | 随 CubeCLT | `/opt/ST/STM32CubeCLT_1.22.0/STLink-gdb-server/bin/ST-LINK_gdbserver` |
| STM32CubeProgrammer CLI | 2.23.0 | `/opt/ST/STM32CubeCLT_1.22.0/STM32CubeProgrammer/bin/STM32_Programmer_CLI` |
| SVD 文件 | 含 `STM32C542.svd` | `/opt/ST/STM32CubeCLT_1.22.0/STMicroelectronics_CMSIS_SVD/` |
| CLion | 2026.1.1（文档同时适用于 2026.2） | `/Applications/CLion.app` |
| OpenOCD（Homebrew） | 0.12.0+dev | **没有 `stm32c5x.cfg`**，不能用于 C5 下载/调试（见第 8 节） |
| Cube CLI | 可选 | `/Users/peakors/.local/stm32cube/bin/cube` |

## 4. 目录结构与命名

```text
.
├── AGENTS.md              # 本文件
├── README.md              # 项目门面：速查表 + 文档/模板导航
├── docs/                  # 中文文档，NN-主题.md
├── templates/             # 可复制的配置模板（OpenOCD、CLion 调试参数等）
├── projects/              # 实际工程（每个工程一个子目录，尚未创建）
└── .gitignore
```

- 新增文档：`docs/NN-主题.md`，`NN` 两位递增，中文标题，并在 `docs/README.md` 登记。
- 新增工程：`projects/<工程名>/`，工程名只用字母、数字、下划线，**不含空格与中文**（CubeMX2 与 CMake 对路径敏感）。
- 新增模板：`templates/<类别>/`，模板里不要放绝对路径，用 `<CubeCLT>`、`<PROJECT>` 之类的占位符。

## 5. 文档写作规范

- 语言：**简体中文**；专有名词保留英文原文（如 Pinout、Pack Manager、CPU Cache）。
- 每篇文档开头三行元信息：`适用版本` / `前置条件` / `验证方式`。
- 界面操作写成 `菜单 > 子菜单 > 按钮`；命令放在 `bash` 代码块里，可直接复制。
- 每个关键步骤给出**预期结果**（如“LD1 以 1 Hz 闪烁”“终端出现 `[OK]`”）。
- 截图放在 `docs/images/<文档名>/`，文件名用英文短横线；不要提交超过 500 KB 的图片（必要时压缩）。
- 引用外部资料用 Markdown 链接，并在文末「参考资料」列出。

## 6. 代码与工程规范

- **CubeMX2 生成的文件不要手改**：`main.c` 只在 `/* USER CODE BEGIN */` 与 `/* USER CODE END */` 之间写代码。
- 新增源文件放到工程内自建目录（如 `User/`），并加入 `CMakeLists.txt`；若 CMake 文件是生成的，改 `.ioc2` 的工程设置或使用 `CMakeLists.txt` 的模板保留区（如生成器支持）。
- 命名：函数 `模块_动作()`（如 `led_init()`、`uart_send_string()`），宏全大写，全局变量加模块前缀。
- 中断处理保持短小，业务逻辑放主循环或状态机。
- 编译必须 **0 error**；新增 warning 需在提交说明里解释。

## 7. 构建 / 烧录 / 调试命令

```bash
# 构建（在工程目录内，CubeMX2 生成 CMake 工程后）
export PATH="/opt/ST/STM32CubeCLT_1.22.0/CMake/bin:$PATH"
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j

# 烧录 ELF（SWD，STLINK-V3EC）
/opt/ST/STM32CubeCLT_1.22.0/STM32CubeProgrammer/bin/STM32_Programmer_CLI \
  -c port=SWD freq=4000 -w build/<PROJECT>.elf -v -rst

# 手动起 GDB server（CLion 会代劳，排查时可单跑）
cd /tmp && /opt/ST/STM32CubeCLT_1.22.0/STLink-gdb-server/bin/ST-LINK_gdbserver \
  -cp /opt/ST/STM32CubeCLT_1.22.0/STM32CubeProgrammer/bin -d -p 61234

# 串口（VCP = USART2 PA2/PA3，115200 8N1）
ls /dev/cu.usbmodem* && screen /dev/cu.usbmodemXXXX 115200
```

> 输出文件名、构建目录以 CubeMX2 实际生成结果为准；上例是约定，不是生成器保证。

## 8. 已知限制

- **OpenOCD 不支持 STM32C5**：Homebrew 0.12.0 与上游 master 均无 `tcl/target/stm32c5x.cfg`。C5 的下载/调试请用 ST-LINK GDB server 或 STM32CubeProgrammer；`templates/openocd/nucleo-c542rc.cfg` 仅作前瞻模板。
- **CubeMX2 重新生成不合并 CMake 改动**：手工改过 `CMakeLists.txt` 后再次生成会被覆盖/备份，务必把改动落到 `.ioc2` 或独立文件。
- **CubeMX2 创建工程时不能选 pack 版本**（默认最新）；C5 若需固定 v2.0.0 pack，官方提供脚本（见 CubeMX2 Release limitations）。
- **CubeMX2 要求 CMake ≥ 3.30**（本机 CubeCLT 为 4.3.1，满足）。
- **CubeMX2 对 C5 有单独限制清单**：<https://dev.st.com/stm32cube-docs/stm32c5-mcu-package/latest/en/stm32cubemx2-config-limitations.html>。
- 本仓库文档基于官方资料整理，**尚未逐屏截图实测**；首次实操后请把截图与偏差补进来。

## 9. 事实来源

| 事实 | 来源 |
| --- | --- |
| 板卡特性、LED/按键/VCP/电源、STLINK-V3EC、256 KB Flash / 64 KB RAM、HSE 24 MHz、LD1=PA5、B1=PC13、BOOT0=PH2 | UM3615《STM32C5x2 Nucleo-64 board user manual》Rev 1（2026-02），<https://www.st.com/resource/en/user_manual/um3615-stm32c5.pdf> |
| C5 系列 Cortex-M33 @144 MHz、CoreMark 593 | STM32C5 系列页 <https://www.st.com/en/microcontrollers-microprocessors/stm32c5-series.html> |
| UART2/PA2/PA3 接 VCP、SB12/SB26/SB46 配置 | UM3615 表 11 |
| STM32CubeMX2 界面与生成流程 | STM32CubeMX2 官方文档 1.1.2，<https://dev.st.com/stm32cube-docs/stm32cubemx2/1.1.2/en/index.html> |
| CLion 的 STM32 集成、ST-LINK GDB server 配置 | JetBrains CLion 文档，<https://www.jetbrains.com/help/clion/embedded-stm32.html> |

## 10. 提交前检查清单

- [ ] 文档中的路径、版本号与第 3 节一致，或已更新第 3 节。
- [ ] 没有提交 `build/`、`misc/`、`*.elf`、`*.bin`、`.DS_Store`。
- [ ] 没有手改生成文件；变更来自 `.ioc2` 或 `USER CODE` 区。
- [ ] 新增文档已在 `docs/README.md` 登记。
- [ ] 命令在本机可执行（至少 `--help` / `--version` 通过）。
- [ ] 提交信息遵循 `type(scope): summary`（如 `docs(cubemx2): 补充时钟配置步骤`）。

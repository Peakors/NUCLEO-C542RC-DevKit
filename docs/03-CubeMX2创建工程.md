# 03 · 用 STM32CubeMX2 创建工程（点灯）

> 适用版本：STM32CubeMX2 1.1.2 · 板卡 NUCLEO-C542RC（STM32C542RCT6）
> 前置条件：已完成 [02-环境搭建](02-环境搭建.md)，CubeMX2 能正常启动并下载 packs
> 验证方式：生成 CMake 工程后能编译出 `.elf`（第 7 节），下载后 LD1 以 1 Hz 闪烁

## 0. 与经典 CubeMX 的区别

| 项目 | 经典 CubeMX 6.x | **CubeMX2 1.x** |
| --- | --- | --- |
| 工程文件 | `.ioc` | **`.ioc2`** |
| 代码来源 | 内置 HAL 拷贝到工程 | 基于 **CMSIS Pack**（Pack Manager 管理） |
| 生成格式 | 多种 IDE / Makefile | **IAR / CMake / Open-CMSIS** |
| CMake 要求 | 无 | **≥ 3.30** |

本仓库统一走 **CubeMX2 + CMake**。

## 1. 新建 Board 工程

1. 启动 CubeMX2，在 **Home** 页选择 **Board**。
2. 搜索框输入 `NUCLEO-C542RC`，选中板卡，点击 **Continue**。
   > ⚠️ 未实测：本机未安装 CubeMX2，板卡是否出现在列表中未实际确认。若搜不到，说明当前 CubeMX2 版本或 C5 pack 未安装，先在 **Pack Manager** 里安装 STM32C5 系列包。
3. 填写 **Project name**（只能用字母、数字、下划线，不能有空格和中文，例如 `blink_led`）与 **Project location**（例如 `~/STM32Project/NUCLEO-C542RC-DevKit/projects`）。
4. 点击 **Automatically Download, Install & Create Project**，等待进度条完成。

预期：进入工程视图，左侧栏出现 Home / Project settings / Pinout / Clock / Peripherals 等图标。

## 2. 工程设置（Project settings）

打开 **Project settings**，确认：

- **Project details**：工程名、路径、器件（应为 STM32C542RCT6）。
- **Packs details used in the project**：显示当前 pack 版本；有新版本时可下拉选择后点 **Update**。
- **Global services**：`HAL System Init`（系统初始化）与 `HAL Common definitions` 在这里配置。
- **IDE project generation**：留到第 6 步。

> 注意：CubeMX2 创建工程时**不能指定 pack 版本**（默认最新）。若 C5 需要固定 v2.0.0 pack，官方提供了转换脚本（见 CubeMX2 Release limitations 页面）。

## 3. 配置引脚

1. 打开 **Pinout** 视图，搜索 `PA5`。
2. 右键 PA5 → **Assign signal > GPIO**。
3. 右键 PA5 → **Configure GPIO**，在 **Pins** 里把 PA5 的 **mode** 改为 **output**。
4. 同理把 `PC13` 配置为 **input**（用户按键，外部下拉，按下为高）；如需中断，在 NVIC 里打开对应 EXTI。

预期：Pinout 图上 PA5 显示为绿色 GPIO_Output，PC13 显示为 GPIO_Input。

> 也可以直接用 **Table view** 批量配置引脚，效果相同。

## 4. 配置时钟

1. 打开 **Clock** 视图。
2. **HSE** 选择板载 24 MHz 晶振（Crystal/Ceramic Resonator）。
3. 目标 SYSCLK 设为 **144 MHz**，让工具自动计算 PLL 参数；或按官方推荐的 PLL 配置手工填写。
4. 检查 USART2、USB 等外设的时钟是否落在合法范围（USB 需要 48 MHz）。

预期：时钟树上没有红色冲突标记，**CPU clock** 显示 144 MHz。

> ⚠️ 具体 PLL 分频/倍频值以 CubeMX2 时钟树的可选值为准，不同 pack 版本可能不同；以工具不报错为通过标准。

## 5. 生成前的检查

- [ ] 器件是 STM32C542RCT6
- [ ] PA5 = GPIO_Output，PC13 = GPIO_Input
- [ ] SYSCLK = 144 MHz，HSE = 24 MHz
- [ ] 没有未解决的冲突（工具会标红并给出提示）

## 6. 生成 CMake 工程

1. 在 **Project settings > IDE project generation**：
   - **Format**：选择 **CMake**
   - **Toolchain**：选择 **GCC**
   - **Destination**：默认工程根目录即可
   - **External sources**：保持默认 **Include fresh packs**（工程自包含、可移动），或选 **Include packs from local** 复用本地 pack
2. 保持生成目录默认值（官方提示：改目录容易出问题）。
3. 点击 **Generate**（或工具栏的齿轮按钮）。

预期：工程目录下出现 `CMakeLists.txt`、`.ioc2`、`Core/`（或等价的生成目录）、`misc/`（packs）等。

> ⚠️ 未实测：生成产物的具体目录名随 CubeMX2 版本与 pack 变化，以实际结果为准；`misc/` 已在本仓库 `.gitignore` 中忽略。

## 7. 编译验证（CLI）

```bash
cd <工程目录>
export PATH="/opt/ST/STM32CubeCLT_1.22.0/CMake/bin:/opt/ST/STM32CubeCLT_1.22.0/Ninja/bin:$PATH"
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j
```

预期：`build/` 下生成 `.elf`（文件名通常与工程名一致）。

## 8. 在 main 里写点灯代码

打开生成的 `main.c`，只在 `USER CODE` 区内添加（注意 CubeMX2 也会生成类似的保留区，按实际生成的注释标记来）：

```c
/* USER CODE BEGIN 2 */
/* USER CODE END 2 */

while (1)
{
  /* USER CODE BEGIN 3 */
  HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);  /* LD1 = PA5，高电平点亮 */
  HAL_Delay(500);
  /* USER CODE END 3 */
}
```

> 函数名以 CubeMX2 实际生成的 HAL2 API 为准；若 pack 使用 HAL2，个别 API 名称可能不同，先看生成的 `main.c` 与头文件。

## 9. 重新生成代码时的冲突处理

CubeMX2 再次生成时，对**用户文件**、**工程文件**（如 `CMakeLists.txt`）、**pack 文件**可分别设置：

| 策略 | 行为 |
| --- | --- |
| Show conflicting files | 逐个弹窗确认 |
| Don't generate | 保留现有文件不动 |
| Backup | 备份后覆盖 |
| Overwrite | 直接覆盖 |

**重点**：CubeMX2 **不会自动合并**你在 `CMakeLists.txt` 里的手工修改。要加源文件，优先改 `.ioc2` 里的工程设置，或把代码放进独立文件并在生成配置中登记。

## 10. 下一步

- 在 CLion 里打开并调试：[04-CLion导入编译调试](04-CLion导入编译调试.md)
- 命令行下载：[05-烧录与命令行调试](05-烧录与命令行调试.md)

## 参考资料

- CubeMX2 快速上手（LED toggle 例子）：<https://dev.st.com/stm32cube-docs/stm32cubemx2/1.1.2/en/docs/markup/CubeMX2_How_To_Quickly_Start.html>
- IDE project generation：<https://dev.st.com/stm32cube-docs/stm32cubemx2/1.1.2/en/docs/markup/CubeMX2_UserManual/CubeMX2_UserManual_IDE_ProjectGen.html>
- Release limitations（pack 版本 / 重新生成限制）：<https://dev.st.com/stm32cube-docs/stm32cubemx2/1.1.2/en/docs/markup/CubeMX2_Release_Limitations.html>

# CLion ST-LINK 调试配置模板（NUCLEO-C542RC）

CLion 2026.1/2026.2 的 **ST-LINK** 调试模板参数。  
路径中 `<CubeCLT>` = `/opt/ST/STM32CubeCLT_1.22.0`，`<PROJECT>` = 工程名。

## 打开位置

| CLion 版本 | 路径 |
| --- | --- |
| 2026.2+ | `Settings > Build, Execution, Deployment > Debugger > Debug Profiles` → **ST-LINK** |
| 2026.1 | `Settings > Build, Execution, Deployment > Debugger > Debug Servers` → `+` → **ST-LINK** |

2026.1 若没有 Debug Servers 页：`Settings > Advanced Settings > Debugger` → 勾选 **Enable Debug Servers**。

## 字段

| 字段 | 值 |
| --- | --- |
| Name | `NUCLEO-C542RC ST-LINK` |
| ST-LINK GDB server | `<CubeCLT>/STLink-gdb-server/bin/ST-LINK_gdbserver` |
| Programmer path（`-cp`） | `<CubeCLT>/STM32CubeProgrammer/bin` |
| Interface | SWD |
| Device | `STM32C542RCT6` |
| Debugger | `<CubeCLT>/GNU-tools-for-STM32/bin/arm-none-eabi-gdb` |
| Executable | `build/debug/<PROJECT>.elf` |
| Download | Always |
| Port | `61234` |

## 说明

- `Programmer path` 必须指向包含 `STM32_Programmer_CLI` 的目录，否则 GDB server 启动即退出。
- 连接不稳定时降低频率：GDB server 参数加 `--frequency 1000`。
- 需要 SWV/SWO 时加 `-z <port> -a <cpu clock>`。
- SVD：`<CubeCLT>/STMicroelectronics_CMSIS_SVD/STM32C542.svd`，用于 Peripherals 视图。
- 本仓库**不使用 OpenOCD**，不要创建 OpenOCD Download & Run 配置。

## 命令行等价（仅排查用）

```bash
cd /tmp && <CubeCLT>/STLink-gdb-server/bin/ST-LINK_gdbserver \
  -cp <CubeCLT>/STM32CubeProgrammer/bin -d -p 61234

<CubeCLT>/GNU-tools-for-STM32/bin/arm-none-eabi-gdb build/debug/<PROJECT>.elf \
  -ex "target remote :61234" -ex "load" -ex "monitor reset run"
```

> ⚠️ 未实测：字段按 CLion Debug servers 文档与 `ST-LINK_gdbserver --help` 整理；首次连接后请把实际可用的组合补回本文件。

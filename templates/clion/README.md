# CLion 调试配置模板（NUCLEO-C542RC）

CLion 2026.1/2026.2 中为 CubeMX2 生成的 CMake 工程手动创建 **Embedded GDB Server** 配置时使用。

## 配置项

| 字段 | 值 |
| --- | --- |
| Type | Embedded GDB Server |
| Name | `NUCLEO-C542RC Debug` |
| Target | 工程 CMake target（通常与工程同名） |
| Executable | `build/debug/<PROJECT>.elf` |
| Debugger | `/opt/ST/STM32CubeCLT_1.22.0/GNU-tools-for-STM32/bin/arm-none-eabi-gdb` |
| GDB server | `/opt/ST/STM32CubeCLT_1.22.0/STLink-gdb-server/bin/ST-LINK_gdbserver` |
| GDB server arguments | `-cp /opt/ST/STM32CubeCLT_1.22.0/STM32CubeProgrammer/bin -d -p 61234` |
| `target remote` args | `tcp:localhost:61234` |
| Download executable | Always |

## 说明

- `-cp` 必须指向包含 `STM32_Programmer_CLI` 的目录，否则 GDB server 启动即退出。
- `-d` 启用 SWD；需要 SWV/SWO 时再加 `-z <port> -a <cpu clock>`。
- 端口 `61234` 可换成任意空闲端口，需与 `target remote` 保持一致。
- 连接不稳定时降低 `STM32CubeProgrammer` 的 `freq`，或在 GDB server 参数里加 `--frequency 1000`。
- SVD：`/opt/ST/STM32CubeCLT_1.22.0/STMicroelectronics_CMSIS_SVD/STM32C542.svd`（用于 Peripherals 视图）。

## 命令行等价

```bash
cd /tmp && /opt/ST/STM32CubeCLT_1.22.0/STLink-gdb-server/bin/ST-LINK_gdbserver \
  -cp /opt/ST/STM32CubeCLT_1.22.0/STM32CubeProgrammer/bin -d -p 61234

/opt/ST/STM32CubeCLT_1.22.0/GNU-tools-for-STM32/bin/arm-none-eabi-gdb build/<PROJECT>.elf \
  -ex "target remote :61234" -ex "load" -ex "monitor reset run"
```

> ⚠️ 未实测：参数按 `ST-LINK_gdbserver --help` 与 CLion 文档整理，首次连接后请把实际可用的组合补回本文件。

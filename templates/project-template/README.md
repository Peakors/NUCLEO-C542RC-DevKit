# 新工程检查清单（NUCLEO-C542RC）

复制本文件到 `projects/<工程名>/README.md`，按顺序勾选。工程名只用小写字母、数字、下划线。

## 1. 工程信息

- [ ] 工程名：`________________`
- [ ] 一句话目标：`________________`
- [ ] 用到的外设：`________________`

## 2. CubeMX2 配置

- [ ] 从 **Board → NUCLEO-C542RC** 创建（不要从 MCU 裸选，避免漏掉板级默认值）
- [ ] HSE = 24 MHz 晶振
- [ ] SYSCLK = 144 MHz
- [ ] USART2 启用（PA2/PA3，115200 8N1），用于 VCP
- [ ] PA5 = GPIO_Output（LD1，高电平点亮）
- [ ] PC13 = GPIO_Input（B1，**禁止配置为输出**）
- [ ] 需要中断时，在 NVIC 中使能对应 EXTI
- [ ] 生成格式 = **CMake**，Toolchain = **GCC**

## 3. 工程结构

```text
projects/<工程名>/
├── README.md            # 本文件
├── CMakeLists.txt       # CubeMX2 生成，不要手改
├── *.ioc2               # CubeMX2 工程文件，必须提交
├── Core/                # 生成代码，USER CODE 区以外不要改
├── User/                # 自建目录：业务代码（可选）
└── build/               # 构建产物，已被 .gitignore 忽略
```

- [ ] `.ioc2` 已提交
- [ ] `build/`、`misc/`、`*.elf` 未提交
- [ ] 自定义源文件通过 `.ioc2` 或生成器可识别的配置加入，不改生成的 `CMakeLists.txt`

## 4. 代码

- [ ] 业务代码在 `USER CODE` 区或 `User/`
- [ ] 函数命名 `模块_动作()`，宏全大写
- [ ] 中断服务程序短小，重活放主循环
- [ ] 编译 0 error

## 5. 验证

- [ ] `cmake --build build -j` 通过
- [ ] `STM32_Programmer_CLI -c port=SWD -w build/<工程名>.elf -v -rst` 下载成功
- [ ] 板上现象：`________________`（如 LD1 1 Hz 闪烁）
- [ ] 串口输出（如有）：`________________`
- [ ] CLion 调试能停在 `main`，Peripherals 能看到寄存器

## 6. 文档

- [ ] 本 README 的「工程信息」已填写
- [ ] 引脚占用表（见下）已更新
- [ ] 现象与截图（可选，放 `docs/images/`）

## 引脚占用

| 引脚 | 功能 | 备注 |
| --- | --- | --- |
| PA5 | LD1 | 高电平点亮 |
| PC13 | B1 | 输入，外部下拉 |
| PA2 | USART2_TX | VCP |
| PA3 | USART2_RX | VCP |
|  |  |  |

# projects/

实际工程目录。每个子目录是一个独立的 CubeMX2 + CMake 工程。

## 约定

- 目录名：小写字母、数字、下划线（如 `blink_led`、`uart_echo`）。
- 每个工程必须有 `README.md`（由 [templates/project-template/README.md](../templates/project-template/README.md) 复制而来）。
- 必须提交：`.ioc2`、生成代码、工程自己的源文件。
- 不要提交：`build/`、`misc/`、`*.elf`、`*.bin`、`*.hex`（已由根目录 `.gitignore` 处理）。

## 新增工程的步骤

```bash
# 1. 建目录并复制检查清单
mkdir -p projects/blink_led
cp templates/project-template/README.md projects/blink_led/README.md

# 2. 用 CubeMX2 在 projects/blink_led 下生成工程
# 3. 编译 + 下载验证
cd projects/blink_led
export PATH="/opt/ST/STM32CubeCLT_1.22.0/CMake/bin:/opt/ST/STM32CubeCLT_1.22.0/Ninja/bin:$PATH"
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j

# 4. 提交
git add projects/blink_led
git commit -m "feat(projects): add blink_led example"
```

## 现有工程

| 工程 | 说明 | 状态 |
| --- | --- | --- |
| （暂无） | — | — |

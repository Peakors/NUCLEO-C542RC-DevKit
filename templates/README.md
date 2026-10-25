# 模板目录

可复制的配置与检查清单。模板中的绝对路径以 `<CubeCLT>`（默认 `/opt/ST/STM32CubeCLT_1.22.0`）和 `<PROJECT>` 占位，复制后请替换。

| 模板 | 用途 | 状态 |
| --- | --- | --- |
| [clion/README.md](clion/README.md) | CLion 内置 ST-LINK 调试参数 | ✅ 可用（待实机复核） |
| [project-template/README.md](project-template/README.md) | 新工程建设检查清单 | ✅ 可用 |

## 使用方式

```bash
# 1. 复制检查清单，放到新工程目录里
mkdir -p projects/blink_led
cp templates/project-template/README.md projects/blink_led/README.md

# 2. 按清单用 CubeMX2 生成工程（生成文件与 README 同级）
# 3. 按 templates/clion/README.md 配调试
```

## 约定

- 模板不放绝对路径，统一用占位符。
- 模板改动后，同步检查 [docs](../docs/README.md) 中引用它的页面。

---
outline: deep
---

# config-file

## 用途

从本地文件读取配置文本并解析为 `Map`（支持 TOML/JSON/YAML）。

## 驱动名

- `file`

## 参数

- `file` / `path` / `config`：配置文件路径
- `format`：`toml`、`json` 或 `yaml`（不传则按扩展名/内容自动识别）

## 示例

```bash
./app --driver=file --file=config.toml
```

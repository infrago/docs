---
outline: deep
---

# config

## 职责

`config` 模块负责解析配置来源并返回 `Map`，供运行时在 `Load` 阶段分发给所有模块。

## 解析优先级

1. 环境变量：`INFRAGO_*`
2. 命令行参数：`--key=value` 或 `--key value`
3. 驱动读取：如 `file` / `redis`

环境变量和命令行参数会做 key 规范化，支持 `INFRAGO_CONFIG_FILE` -> `file`、`INFRAGO_REDIS_ADDR` -> `addr` 这类别名。

`config.Redact(Map)` 可用于输出调试信息前脱敏，默认会隐藏 `password`、`secret`、`token`、`key`、`api_key` 等字段。

## 默认行为

- 默认驱动：`default(file)`
- 默认文件候选：`config.toml`、`config.json`、`config.yaml`、`config.yml`、`{process}.toml`、`{process}.json`、`{process}.yaml`、`{process}.yml`

## 常用启动方式

```bash
# 直接给配置文件
./app config.toml

# 指定驱动与参数
./app --driver=redis --server=127.0.0.1 --database=0 --key=infrago-config

# 使用别名
INFRAGO_CONFIG_FILE=config.yaml ./app
```

## 相关驱动

- [config-file](/zh/drivers/config-file)
- [config-redis](/zh/drivers/config-redis)

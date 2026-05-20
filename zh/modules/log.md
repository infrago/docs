---
outline: deep
---

# log

## 职责

统一日志等级、格式、缓冲与多驱动输出。

## 配置结构

```toml
[log.console]
driver = "default"
levels = ["trace"]
buffer = 4096
batch = 1024
timeout = "100ms"
overflow = "block"

[log.error]
driver = "file"
levels = ["error", "panic", "fatal"]
json = true
sample = 1
buffer = 8192
batch = 2048
timeout = "150ms"
overflow = "block"

[log.error.setting]
store = "store/log"
output = "error.log"
maxsize = "128MB"
slice = "day"
compress = true
```

核心字段：

- `driver`
- `level` / `levels`
- `json`
- `sample`（`0~1`）
- `unsafe_fields`：跳过结构化字段拷贝的高性能模式；仅在确认调用方不会在写入后修改 `fields` 时启用
- `buffer` / `batch` / `timeout`
- `overflow`：`block`（默认）/ `drop` / `drop_newest` / `drop_oldest`
- `drop`：`old`（默认）/ `new`（仅在 `overflow = "drop"` 时生效）
- `flag` / `format`
- `setting`

每条日志会自动附带 `project`、`profile`、`node` 三个字段（结构化字段）。

## 对外 API

- `log.Debug/Trace/Info/Notice/Warning/Error/Panic/Fatal`（灵活参数）
- `log.Debugf/Tracef/...`（格式化）
- `log.Debugw/Tracew/...`（结构化）
- `log.Write(level, args...)`
- `log.Writef(level, format, args...)`
- `log.Writew(level, body, fields)`
- `log.Levels()`
- `log.Stats()`
- `log.SetExitFunc(fn)`：替换 `Fatal/Fatalf/Fatalw` 使用的退出函数，主要用于测试；返回值可恢复旧函数

高频路径建议优先使用 `*f` 或 `*w` 形式，`Info(args...)` 这类灵活参数会做格式串识别；模块会缓存识别结果以降低重复格式的开销。

`Stats()` 可用于背压与降级调优，包含：

- `queue_len`、`queue_cap`
- `drop_count`、`last_drop_unix`
- `write_error_count`、`last_error`
- `queued_count`、`sync_write_count`、`sync_fallback_count`
- `flush_count`、`flush_log_count`
- `last_flush_latency_ms`、`last_dispatch_latency_ms`、`avg_dispatch_latency_ms`
- `connections`：按日志实例返回连接级指标，包括 `queue_len`、`queue_cap`、`overflow`、`queued_count`、`drop_count`、`write_error_count`、`last_error`、`last_write_latency_ms`、`avg_write_latency_ms`

## 驱动

- `default`（stdout/stderr）
- [log-file](/zh/drivers/log-file)
- [log-greptime](/zh/drivers/log-greptime)

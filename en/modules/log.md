---
outline: deep
---

# log

Unified logging module with async queue, batch flush, sampling, and pluggable drivers.

## Config

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

Common fields:

- `driver`
- `level` or `levels`
- `json`
- `sample` (`0~1`)
- `buffer` / `batch` / `timeout`
- `overflow`: `block` (default) / `drop` / `drop_newest` / `drop_oldest`
- `drop`: `old` (default) / `new` (used when `overflow = "drop"`)
- `flag` / `format`
- `setting`

Each log record automatically carries `project`, `profile`, `node` in structured fields.

## API

Three styles are supported for each level:

- flexible: `Info(args...)`
- format: `Infof(format, args...)`
- structured: `Infow(body, fields)`

Also available:

- `Write(level, args...)`
- `Writef(level, format, args...)`
- `Writew(level, body, fields)`
- `Levels()`
- `Stats()`
- `SetExitFunc(fn)`: replaces the exit function used by `Fatal/Fatalf/Fatalw`, mainly for tests; the returned function restores the previous exit function

`Stats()` returns runtime metrics for backpressure/degradation tuning:

- `queue_len`, `queue_cap`
- `drop_count`, `last_drop_unix`
- `write_error_count`, `last_error`
- `queued_count`, `sync_write_count`, `sync_fallback_count`
- `flush_count`, `flush_log_count`
- `last_flush_latency_ms`, `last_dispatch_latency_ms`, `avg_dispatch_latency_ms`
- `connections`: per log instance metrics, including `queue_len`, `queue_cap`, `overflow`, `queued_count`, `drop_count`, `write_error_count`, `last_error`, `last_write_latency_ms`, `avg_write_latency_ms`

## Drivers

- `default` (stdout/stderr)
- [log-file](/en/drivers/log-file)
- [log-greptime](/en/drivers/log-greptime)

---
outline: deep
---

# trace

`trace` 是 infrago 的链路追踪模块，支持多连接并行写入（双写/多写）。

## 核心特性

- 内置 `default` 驱动（控制台输出）
- 多实例配置：`[trace.a] [trace.b]` 同时写
- 异步缓冲 + 批量刷写（`buffer` + `timeout`）
- 采样（`sample`，0~1）
- 与 `Meta` 集成：`ctx.Begin(...)` / `ctx.Trace(...)`
- 自动 span：`http/web/event/queue` 入口自动 begin/end，`Invoke` 自动子 span
- 自动命名：`method:<key>` / `service:<key>` / `trigger:<key>` / `http:<key>` / `web:<key>` / `event:<key>` / `queue:<key>`
- 内置统一属性构建：`infra.TraceAttrs(...)`
- 通过 `TraceHook` 挂接，可被第三方模块替换

## 配置

单实例：

```toml
[trace]
driver = "default"
json = true
sample = 1
buffer = 1024
timeout = "200ms"
format = "%time% [%status%] %name% trace=%traceId% span=%spanId% duration=%durationMs%ms"
```

采样配置放到连接子节：`trace.<conn>.sample`（与 `setting` 同级）：

```toml
[trace.greptime]
driver = "greptime"

[trace.greptime.sample]
rate = 0.2
error = true
key = "trace_id"
rules = [
  { name = "http:*", sample = 0.1 },
  { name = "service:*", sample = 0.3 },
  { name = "service:user.*", sample = 1, attrs = { vip = true } }
]
```

多实例双写：

```toml
[trace.console]
driver = "default"
json = true

[trace.greptime]
driver = "greptime"
fields = { trace_id = "tid", span_id = "sid", parent_span_id = "psid", timestamp = "ts" }
[trace.greptime.setting]
host = "127.0.0.1"
port = 4001
database = "public"
table = "traces"
```

字段映射配置在连接本身（`trace.<conn>.fields`），不放在 `setting` 下。

## 使用方式

### 1. 手动 span

```go
span := ctx.Begin("service:user.login", infra.TraceAttrs("user", infra.TraceKindServer, "login", base.Map{
  "module": "http",
  "operation": "serve",
}))
defer span.End()
```

### 2. 事件追踪

```go
_ = ctx.Trace("user.login.validated", base.Map{
  "status": "ok",
  "userId": 1001,
})
```

`status=error` 时可带 `error` 字段：

```go
_ = ctx.Trace("user.login.failed", base.Map{
  "status": "error",
  "error":  "invalid password",
})
```

## Span 字段（OTel 对齐）

- `trace_id` / `span_id` / `parent_span_id`
- `name`
- `kind`（`internal/server/client/producer/consumer`）
- `service_name` / `target`
- `status`（`ok/error`）+ `status_code` + `status_message`
- `duration_ms` / `start_ms` / `end_ms`
- `start_time_unix_nano` / `end_time_unix_nano`
- `timestamp`
- `attributes` / `resource`
- 运行时维度：`project` / `profile` / `node`（在参数表与 resource 中可用）

## 统一属性建议

- 固定字段：`service`、`kind`、`target`
- 推荐扩展：`module`、`operation`
- 按需扩展：`connection`、`site`、`method`、`path`、`attempt` 等

## 默认驱动说明

`trace` 包内置 `default` 驱动，不依赖外部存储，直接输出到控制台。

- `status=error` 输出到 `stderr`
- 其它输出到 `stdout`

## 驱动

- [trace-greptime](/zh/drivers/trace-greptime)
- [trace-file](/zh/drivers/trace-file)
- [trace-otlp](/zh/drivers/trace-otlp)

## 运行指标

可通过 `trace.Stats()` 查看：

- `queue_len` / `queue_cap`
- `queued_count`
- `sync_fallback_count`
- `flush_count` / `flush_span_count`
- `write_error_count`
- `dropped_count`
- `dynamic_sample_factor`

---
outline: deep
---

# trace

`trace` is infrago's tracing module with multi-connection fanout (dual-write/multi-write).

## Features

- Built-in `default` driver (console output)
- Multi-instance config: `[trace.a] [trace.b]` writes to all
- Async buffer + batch flush (`buffer` + `timeout`)
- Sampling (`sample`, 0~1)
- `Meta` integration: `ctx.Begin(...)` / `ctx.Trace(...)`
- Auto span: `http/web/event/queue` entry auto begin/end, and `Invoke` creates child spans
- Auto naming: `method:<key>` / `service:<key>` / `trigger:<key>` / `http:<key>` / `web:<key>` / `event:<key>` / `queue:<key>`
- Built-in normalized attrs helper: `infra.TraceAttrs(...)`
- `TraceHook` based; replaceable by third-party modules

## Config

Single instance:

```toml
[trace]
driver = "default"
json = true
sample = 1
buffer = 1024
timeout = "200ms"
format = "%time% [%status%] %name% trace=%traceId% span=%spanId% duration=%durationMs%ms"
```

Sampling config lives under connection sub-section: `trace.<conn>.sample` (same level as `setting`):

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

Dual-write:

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

Field mapping is configured on the connection itself (`trace.<conn>.fields`), not under `setting`.

## Usage

### 1) Manual span

```go
span := ctx.Begin("service:user.login", infra.TraceAttrs("user", infra.TraceKindServer, "login", Map{
  "module": "http",
  "operation": "serve",
}))
defer span.End()
```

### 2) Trace event

```go
_ = ctx.Trace("user.login.validated", Map{
  "status": "ok",
  "userId": 1001,
})
```

With error status:

```go
_ = ctx.Trace("user.login.failed", Map{
  "status": "error",
  "error":  "invalid password",
})
```

## Span Fields (OTel-aligned)

- `trace_id` / `span_id` / `parent_span_id`
- `name`
- `kind` (`internal/server/client/producer/consumer`)
- `service_name` / `target`
- `status` (`ok/error`) + `status_code` + `status_message`
- `duration_ms` / `start_ms` / `end_ms`
- `start_time_unix_nano` / `end_time_unix_nano`
- `timestamp`
- `attributes` / `resource`
- runtime dimensions: `project` / `profile` / `node` (available in field mapping and resource)

## Normalized Attrs

- Base fields: `service`, `kind`, `target`
- Recommended fields: `module`, `operation`
- Optional fields by module: `connection`, `site`, `method`, `path`, `attempt`

## Built-in Default Driver

`trace` package includes `default` driver in-module.

- `status=error` -> `stderr`
- others -> `stdout`

## Drivers

- [trace-greptime](/en/drivers/trace-greptime)
- [trace-file](/en/drivers/trace-file)
- [trace-otlp](/en/drivers/trace-otlp)

## Runtime Metrics

Available via `trace.Stats()`:

- `queue_len` / `queue_cap`
- `queued_count`
- `sync_fallback_count`
- `flush_count` / `flush_span_count`
- `write_error_count`
- `dropped_count`
- `dynamic_sample_factor`

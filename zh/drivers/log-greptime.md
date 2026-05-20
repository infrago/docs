---
outline: deep
---

# log-greptime

## 用途

GreptimeDB 日志驱动（ingester 协议）。

## 驱动名

- `greptime`

## setting 参数

- `host` / `server`
- `port`（默认 `4001`）
- `username` / `user`
- `password` / `pass`
- `database` / `db`（默认 `public`）
- `table`（默认 `logs`）
- `timeout`（默认 `5s`）
- `insecure`（默认 `true`）
- `tls`（与 `insecure` 互补）

## 字段编码

`fields` 会写入 Greptime 的 `fields` 字符串列。空字段使用 `{}`；基础标量字段会走轻量编码；如果调用方已经准备好 JSON，可传入单字段 `{"_json": "..."}` 或 `{"__json": "..."}`，驱动会在校验 JSON 合法后直接写入。

## 示例

```toml
[log]
driver = "greptime"

[log.setting]
host = "127.0.0.1"
port = 4001
database = "public"
table = "app_logs"
insecure = true
```

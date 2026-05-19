---
outline: deep
---

# queue-redis

## 用途

Redis 队列驱动，基于 list，支持延迟发布与重试。

消费时会先把消息移动到 `{queue}:processing`，处理完成后再确认删除；进程在处理过程中崩溃时，消息会留在 processing list，并在驱动启动时恢复回主队列。

## 驱动名

- `redis`

## setting 参数

- `addr` / `server` / `host` + `port`
- `username` `password`
- `database`

## 示例

```toml
[queue.default]
driver = "redis"

[queue.default.setting]
addr = "127.0.0.1:6379"
database = 0
```

---
outline: deep
---

# queue-redis

## 用途

Redis 队列驱动，基于 Redis Streams consumer group，支持延迟发布、重试、ACK、崩溃后 pending 消息认领和延迟 bucket。

每个队列使用 `{queue}:stream` 作为 stream key。驱动启动时会创建 consumer group，并用独立 consumer 名处理消息；超过 `claim` 空闲时间的 pending 消息会被其他 worker 认领继续处理。

延迟消息写入 `{queue}:delayed` sorted set，到期后搬入 stream，避免长延迟消息反复占用 stream consumer。

从旧版 list 实现升级时，驱动会在启动时把 `{queue}` 和 `{queue}:processing` 中的旧消息迁移到 `{queue}:stream`。

## 驱动名

- `redis`

## setting 参数

- `addr` / `server` / `host` + `port`
- `username` `password`
- `database`
- `group`（consumer group，默认 `infragoq`）
- `consumer`（consumer 名前缀，默认由主机名、进程号和时间生成）
- `claim`（pending 消息最小空闲认领时间，默认 `5m`；数字按秒处理）
- `consumer_idle`（空闲 consumer 清理阈值，默认 `1h`；数字按秒处理）
- `maxlen`（stream 近似最大长度，设置后使用 `XADD MAXLEN ~`）

## 示例

```toml
[queue.default]
driver = "redis"

[queue.default.setting]
addr = "127.0.0.1:6379"
database = 0
```

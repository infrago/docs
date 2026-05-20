---
outline: deep
---

# queue-nats

## 用途

NATS 队列驱动，支持普通 NATS 与 JetStream，支持延迟和重试。

JetStream 驱动使用 durable consumer，并会从积压消息开始投递，适合需要持久队列语义的场景。

普通 NATS 不持久化消息。延迟发布通过消费端收到后再次发布实现，属于 best-effort；需要可靠延迟、ACK 和崩溃恢复时应使用 JetStream。

JetStream 的延迟发布使用绝对到期时间 header，并通过 `NakWithDelay` 等待到期；durable consumer 名和 queue group 名分离，便于排查订阅状态。

## 驱动名

- `nats`
- `natsjs` / `nats-js` / `jetstream`

## setting 参数

- `url` / `server`
- `token`
- `user` / `username`
- `pass` / `password`
- `stream`（JetStream，默认 `INFRAGOQ`）

## 示例

```toml
[queue.default]
driver = "nats"

[queue.default.setting]
url = "nats://127.0.0.1:4222"
```

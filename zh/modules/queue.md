---
outline: deep
---

# queue

## 职责

异步任务队列，支持延迟发布、重试和并发消费线程。

## 配置结构

```toml
[queue.default]
driver = "redis"
external = false
codec = "gob"
weight = 1
prefix = "prod"

[queue.default.setting]
addr = "127.0.0.1:6379"
database = 0
# default 内存驱动可用；其他驱动忽略
buffer = 256
```

字段：

- `driver`
- `external`
- `codec`（默认 `gob`）
- `weight`
- `prefix`
- `setting`

`default` 内存驱动支持：

- `setting.buffer`：队列 channel 缓冲大小，也可以在单个 `Queue.Setting["buffer"]` 上覆盖。
- `setting.blocking_publish`：队列满时是否阻塞等待。
- `setting.publish_timeout`：发布阻塞超时，例如 `"500ms"`；设置后会启用阻塞等待并在超时后返回队列满错误。

## 组件

- `Queue`（`Thread`、`Retry` 等）
- `Declare`
- `Filter`
- `Handler`

`Retry` 表示最多重试次数对应的延迟序列。超过序列长度后，`Context.Final()` 为 `true`，模块不会继续重新投递。

业务处理 panic 会被捕获并转为可重试失败，不会打死消费 worker。`Context.Context()` / `Context.Done()` 可用于感知模块停止时的取消信号。

如果 `Queue.Setting` 配置 `dead`、`dead_letter` 或 `dlq`，最终失败或坏消息会转发到对应队列。

## 对外 API

- `queue.Publish / PublishTo`
- `queue.DeferredPublish / DeferredPublishTo`

## 驱动

- `default`
- [queue-nats](/zh/drivers/queue-nats)
- [queue-redis](/zh/drivers/queue-redis)

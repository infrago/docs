# queue

Async queue module with delay and retry capabilities.

Components:

- `Queue`, `Declare`, `Filter`, `Handler`

The default in-memory driver supports:

- `setting.buffer`, channel buffer size; an individual `Queue.Setting["buffer"]` can override it.
- `setting.blocking_publish`, whether publish waits when the queue is full.
- `setting.publish_timeout`, publish wait timeout such as `"500ms"`; when set, publish waits and returns a full-queue error on timeout.

`Retry` is the delay sequence and maximum retry count. Once the attempt exceeds the configured sequence, `Context.Final()` is true and the module stops re-delivery.

Panics in queue handlers are recovered and converted to retryable failures so workers keep running. `Context.Context()` and `Context.Done()` expose cancellation when the module stops.

If `Queue.Setting` contains `dead`, `dead_letter`, or `dlq`, final failures and malformed messages are forwarded to that queue.

API:

- `queue.Publish`
- `queue.DeferredPublish`
- `queue.PublishTo`, `queue.DeferredPublishTo`

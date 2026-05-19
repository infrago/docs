# queue

Async queue module with delay and retry capabilities.

Components:

- `Queue`, `Declare`, `Filter`, `Handler`

`Retry` is the delay sequence and maximum retry count. Once the attempt exceeds the configured sequence, `Context.Final()` is true and the module stops re-delivery.

API:

- `queue.Publish`
- `queue.DeferredPublish`
- `queue.PublishTo`, `queue.DeferredPublishTo`

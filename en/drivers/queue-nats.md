# queue-nats

NATS queue drivers. Supports both core NATS and JetStream.

JetStream uses durable consumers and starts with accumulated messages, which is the preferred mode when persistent queue semantics are required.

Core NATS does not persist messages. Delayed publish is implemented by receiving and re-publishing later, so it is best-effort. Use JetStream when reliable delay, ACK, and crash recovery are required.

JetStream delayed publish uses an absolute due-time header and waits with `NakWithDelay`. Durable consumer names and queue group names are separated to make subscription state easier to inspect.

## Drivers

- `nats`
- `natsjs` / `nats-js` / `jetstream`

## Settings

- `url` / `server`
- `token`
- `user` / `username`
- `pass` / `password`
- `stream` for JetStream, default `INFRAGOQ`

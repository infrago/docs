# queue-nats

NATS queue drivers. Supports both core NATS and JetStream.

JetStream uses durable consumers and starts with accumulated messages, which is the preferred mode when persistent queue semantics are required.

## Drivers

- `nats`
- `natsjs` / `nats-js` / `jetstream`

## Settings

- `url` / `server`
- `token`
- `user` / `username`
- `pass` / `password`
- `stream` for JetStream, default `INFRAGOQ`

---
outline: deep
---

# ws

## Responsibility

`ws` is infrago's lightweight real-time client channel module.

It focuses on:

- taking over upgraded WebSocket connections from `http/web.Context.Upgrade()`
- managing `Conn / Session / User / Group`
- defining inbound `Message` and outbound `Command`
- providing `Reply / Push / PushUser / Broadcast / Groupcast`
- providing queueing, backpressure, compression, export, and runtime metrics

It is **not** intended to be a full realtime business platform.

Business orchestration should still primarily flow through `bus`.

## Layering

- `http/web`: routing, auth, args, `Upgrade`
- `ws`: connection, protocol, session, client realtime messages
- `bus`: cross-node fanout

In short:

- `bus` is for module/node messaging
- `ws` is for node-to-client realtime delivery

## Basic Model

### Upgrade Entry

```go
infra.Register(".socket", web.Router{
    Uri:  "/socket",
    Name: "socket",
    Action: func(ctx *web.Context) {
        if err := ctx.Upgrade(); err != nil {
            ctx.Error(infra.Fail.With(err.Error()))
        }
    },
})
```

### Inbound Message

```go
infra.Register("demo.echo", ws.Message{
    Name: "echo",
    Args: base.Vars{
        "text": {Type: "string", Required: true},
    },
    Action: func(ctx *ws.Context) {
        _ = ctx.Reply("demo.echoed", base.Map{
            "text": ctx.Value["text"],
        })
    },
})
```

### Outbound Command

```go
infra.Register("demo.echoed", ws.Command{
    Name: "echoed",
    Args: base.Vars{
        "text": {Type: "string"},
    },
})
```

## Envelope

Default outbound envelope:

```json
{"code":0,"name":"demo.notice","data":{"text":"hello"},"time":1770000000}
```

Fields:

- `code`: `0` means success, non-zero means failure
- `name`: message / command name
- `data`: business payload
- `text`: failure text, usually empty on success
- `time`: server timestamp

Inbound compatibility:

- message name: `name` / `msg`
- payload: `data` / `args`
- when `data/args` is missing, all remaining fields are merged into payload

## Types

- `ws.Message`
  inbound client-to-server message
- `ws.Command`
  outbound server-to-client command

This split is clearer than treating both directions as the same event type.

## Hook / Filter / Handler

- `ws.Hook`
  low-level lifecycle hooks: `Open / Receive / Send / Close`
- `ws.Filter`
  inbound `Message` execution chain
- `ws.Handler`
  error handling: `Invalid / Denied / Error`

To normalize failure replies, a common pattern is:

```go
_ = ctx.Answer("demo.notice", nil, ctx.Result())
```

## Session / User / Group

Internally `ws` tracks:

- `session -> conn`
- `user -> sessions`
- `group -> sessions`

Common APIs:

- `ctx.BindUser(uid)`
- `ctx.Join(group...)`
- `ctx.Leave(group...)`
- `ctx.LeaveAll()`
- `ctx.InGroup(group)`
- `ctx.Groups()`

## Sending APIs

- `ctx.Reply(name, data)`
- `ctx.Answer(name, data, res)`
- `ctx.Push(sid, name, data)`
- `ctx.PushUser(uid, name, data)`
- `ctx.Broadcast(name, data)`
- `ctx.Groupcast(gid, name, data)`

Result-returning variants:

- `ReplyResult`
- `PushResult`
- `PushUserResult`
- `BroadcastResult`
- `GroupcastResult`

These results represent whether the message successfully entered the local send path / queue, not whether the client finally acknowledged it.

## Config

```toml
[ws]
format = "text"
codec = "json"
message_key = "name"
payload_key = "data"
ping_interval = "30s"
read_timeout = "75s"
write_timeout = "10s"
max_message_size = 4194304
queue_size = 128
queue_policy = "close"
compression = false
compress_level = 0
observe_interval = "30s"
observe_log = false
observe_trace = false
```

Key fields:

- `format`: `text` / `binary`
- `codec`: default `json`
- `message_key` / `payload_key`
- `ping_interval` / `read_timeout` / `write_timeout`
- `max_message_size`
- `queue_size` / `queue_policy`
- `compression` / `compress_level`
- `observe_interval` / `observe_log` / `observe_trace`

## Queue and Backpressure

Module-level queue policies:

- `block`
- `close`
- `drop`

`ws.Command.Setting` also supports per-command tuning:

```go
ws.Command{
    Setting: base.Map{
        "priority": "high",      // high | normal | low
        "queue_policy": "block", // optional override
    },
}
```

Default policy behavior:

- `high`: preserved first, defaults to `block`
- `normal`: uses module-level `queue_policy`
- `low`: automatically degraded under pressure, defaults to `drop`

## Protocol Export

`ws.Export()` returns a frontend-friendly document including:

- config
- request/response envelope docs
- `Message / Command` definitions
- sample payloads
- built-in error code descriptions

This is intended to be exposed through a debug or admin endpoint for frontend consumers.

## Metrics

`ws.Metrics()` returns:

- `connections`
- `users`
- `messages_received`
- `messages_sent`
- `receive_failed`
- `send_failed`
- `bytes_received`
- `bytes_sent`
- `avg_receive_bytes`
- `avg_send_bytes`
- `queued`
- `dropped`

When configured with:

- `observe_log = true`
- `observe_trace = true`

the module periodically reports these metrics to `log` / `trace`.

## Design Notes

For infrago, `ws` is best kept as a lightweight realtime channel module rather than immediately turning it into a full realtime platform.

The boundary is best kept around:

- connection
- protocol
- session
- fanout
- observability

while heavier business orchestration stays in:

- `bus`
- or future higher-level realtime modules

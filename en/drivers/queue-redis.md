# queue-redis

Redis-backed queue driver using Redis Streams consumer groups.

Each queue uses `{queue}:stream` as the stream key. The driver creates a consumer group on startup, uses distinct consumer names for workers, and claims pending entries that have been idle longer than `claim`.

Delayed messages are stored in the `{queue}:delayed` sorted set and moved into the stream when due, so long delays do not repeatedly occupy stream consumers.

When upgrading from the previous list-based implementation, startup migrates legacy messages from `{queue}` and `{queue}:processing` into `{queue}:stream`.

## Driver

- `redis`

## Settings

- `addr`, or `server` / `host` + `port`
- `username` / `password`
- `database`
- `group`, consumer group, default `infragoq`
- `consumer`, consumer name prefix; defaults to host, process id, and timestamp
- `claim`, minimum idle time before pending entries can be claimed; default `5m`; numbers are seconds
- `consumer_idle`, idle consumer cleanup threshold; default `1h`; numbers are seconds
- `maxlen`, approximate stream max length using `XADD MAXLEN ~`

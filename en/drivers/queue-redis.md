# queue-redis

Redis-backed queue driver using Redis lists.

Messages are moved to `{queue}:processing` before execution and removed after successful handling or successful retry scheduling. If a worker crashes while handling a message, the message remains in Redis and is restored to the main queue when the driver starts.

## Driver

- `redis`

## Settings

- `addr`, or `server` / `host` + `port`
- `username` / `password`
- `database`

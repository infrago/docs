# log-greptime

GreptimeDB log driver using the ingester protocol.

## Setting

- `host` / `server`
- `port` (default `4001`)
- `username` / `user`
- `password` / `pass`
- `database` / `db` (default `public`)
- `table` (default `logs`)
- `timeout` (default `5s`)
- `insecure` (default `true`)
- `tls` (inverse of `insecure`)

## Fields

Structured `fields` are written to the Greptime `fields` string column. Empty fields use `{}`; primitive scalar fields use a lighter encoder. If JSON is already prepared, pass a single field `{"_json": "..."}` or `{"__json": "..."}` and the driver will validate and write it directly.

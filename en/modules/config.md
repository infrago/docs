# config

Loads runtime configuration through pluggable drivers.

Sources:

- environment variables (`INFRAGO_*`)
- command-line args (`--key=value`)

Environment and CLI keys are normalized. For example, `INFRAGO_CONFIG_FILE` maps to `file`, and `INFRAGO_REDIS_ADDR` maps to `addr`.

Use `config.Redact(Map)` before emitting debug output. It hides sensitive fields such as `password`, `secret`, `token`, `key`, and `api_key`.

Drivers:

- `default/file`
- `config-file`
- `config-redis`

Default file candidates:

- `config.toml`
- `config.json`
- `config.yaml`
- `config.yml`
- `{process}.toml`
- `{process}.json`
- `{process}.yaml`
- `{process}.yml`

Examples:

```bash
./app config.toml
./app --driver=redis --addr=127.0.0.1:6379 --key=infrago-config
INFRAGO_CONFIG_FILE=config.yaml ./app
```

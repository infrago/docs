# config-redis

Reads configuration text from a Redis key and decodes it into a runtime `Map`.

## Driver

- `redis`

## Parameters

- `addr`, or `server`/`host` + `port`
- `username` / `password`
- `database`
- `key` defaults to `infrago-config`
- `format`: optional `toml`, `json`, or `yaml`; when omitted, the driver detects by content
- `timeout`: optional duration such as `3s` or `500ms`; defaults to `3s`
- `tls`: optional `true/false`, plus `tls_server_name` and `tls_insecure_skip_verify`
- `mode=cluster` or `cluster=true` with `cluster_addrs` / `addrs`
- `mode=sentinel` or `sentinel=true` with `master_name` and `sentinel_addrs` / `sentinels`
- Sentinel authentication can use `sentinel_username` and `sentinel_password`

## Example

```bash
./app --driver=redis --addr=127.0.0.1:6379 --key=infrago-config --timeout=3s
./app --driver=redis --mode=cluster --cluster-addrs=redis-a:6379,redis-b:6379 --tls=true
./app --driver=redis --mode=sentinel --master-name=mymaster --sentinel-addrs=sentinel-a:26379,sentinel-b:26379
```

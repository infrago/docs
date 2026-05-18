---
outline: deep
---

# config-redis

## 用途

从 Redis 的一个 key 中读取配置文本并解析为 `Map`。

## 驱动名

- `redis`

## 参数

- `server` `port`
- `username` `password`
- `database`
- `key`（默认 `infrago-config`）
- `addr`，或 `server`/`host` + `port`
- `format`（可选，`toml/json/yaml`）
- `timeout`（可选，如 `3s`、`500ms`，默认 `3s`）
- `tls`（可选，`true/false`）、`tls_server_name`、`tls_insecure_skip_verify`
- `mode=cluster` 或 `cluster=true` + `cluster_addrs` / `addrs`
- `mode=sentinel` 或 `sentinel=true` + `master_name` + `sentinel_addrs` / `sentinels`
- Sentinel 认证可使用 `sentinel_username`、`sentinel_password`

## 示例

```bash
./app --driver=redis --addr=127.0.0.1:6379 --key=infrago-config --timeout=3s
./app --driver=redis --mode=cluster --cluster-addrs=redis-a:6379,redis-b:6379 --tls=true
./app --driver=redis --mode=sentinel --master-name=mymaster --sentinel-addrs=sentinel-a:26379,sentinel-b:26379
```

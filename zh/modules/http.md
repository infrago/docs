---
outline: deep
---

# http

## 职责

单站点 HTTP 模块，提供 Router/Filter/Handler 处理流水线。

## 配置结构

```toml
[http]
driver = "default"
host = "0.0.0.0"
port = 8080
charset = "utf-8"
cookie = "infrago"
token = true
maxage = "24h"
httponly = true
static = "asset/static"
upload = "store/upload"

[http.cross]
enable = true
origins = ["https://admin.example.com"]
```

常用字段：

- `driver` `host` `port` `domain`
- `certfile` `keyfile`
- `charset`
- `cookie` `token` `expire` `crypto` `maxage` `httponly`
- `upload` `static` `defaults`
- `setting`

跨域只支持实例级配置：

- 默认实例：`[http.cross]`
- 命名实例：`[http.xxx.cross]`

## 组件

- `Router`
- `Filter`
- `Handler`
- `Context`（`Text/JSON/View/File/Echo` 等）

## 驱动

- `default`（`gorilla/mux`）

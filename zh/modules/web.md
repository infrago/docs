---
outline: deep
---

# web

## 职责

多站点 Web 模块，支持一个进程内承载多个站点（site）和多域名路由。

## 配置结构

```toml
[web]
driver = "default"
port = 8080
shared = "shared"

[site.admin]
domain = "admin.example.com"
static = "asset/admin"
cross = { enable = true, origins = ["https://console.example.com"] }

[site.api]
domain = "api.example.com"
```

支持两类站点定义：

- `web.site` / `web.sites`
- `site`

跨域只支持站点级配置：`[site.xxx.cross]` 或 `site.xxx.cross = {...}`。

## 组件

- `Router`
- `Filter`
- `Handler`
- `Context`

## URL 能力

- `web.RouteUrl(name, values...)`
- `web.SiteUrl(site, path, options...)`

## 驱动

- `default`（`gorilla/mux`）

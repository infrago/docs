---
outline: deep
---

# data-mysql

`data` 模块 MySQL 驱动。

## 驱动名

- `mysql`

## 配置

```toml
[data]
driver = "mysql"
url = "app:app@tcp(127.0.0.1:3306)/app?parseTime=true&charset=utf8mb4"

[data.setting]
slow = "200ms"
cache = { enable = true, ttl = "3s" }
```

## 连接参数

当前驱动读取：

- `url`（推荐）
- `setting.dsn`（兜底）

## 方言能力

- `ILike`：不支持（内部用 `LOWER(x) LIKE LOWER(?)` 兼容）
- `Returning`：不支持
- JSON 包含：支持（`JSON_CONTAINS`）
- JSON overlap：支持（`JSON_OVERLAPS`）
- JSON path 更新：支持（`JSON_SET`）

## 示例

```go
_ = data.Base().Table("user").Update(Map{
    "$setPath": Map{
        "profile.nickname": "alice",
    },
}, Map{"id": 1001})
```

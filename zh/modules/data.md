---
outline: deep
---

# data

`data` 是 infrago 的统一数据层模块，支持 SQL 与 MongoDB 驱动。

## 核心变化（最新版）

- API 改为结果直返，不直接返回 `error`
- 统一通过 `db.Error()` 获取最后错误
- 新增 `db.Parse(...)`（驱动级解析）
- 支持 `Slice`（直接返回 total + items）
- 支持事务 `Tx`
- 支持只读事务 `TxReadOnly`
- 支持迁移 `Migrate`
- 支持缓存（按表维度失效）
- 支持数据变更 Watcher（`data.watcher`）
- 支持 `WithContext/WithTimeout`（统一超时与取消）
- 支持连接池配置（`maxOpen/maxIdle/maxLifetime/maxIdleTime`）
- 支持连接池观测（`data.GetPoolStats()`）
- Query DSL 解析编译缓存（相同查询入参复用）

## 安装

```bash
go get github.com/infrago/data
go get github.com/infrago/data-postgres
go get github.com/infrago/data-mysql
go get github.com/infrago/data-sqlite
go get github.com/infrago/data-mongodb
```

## 使用风格

```go
db := data.Base()
defer db.Close()

users := db.Table("user").Query(base.Map{"status": "active"})
if db.Error() != nil {
  // handle
}

total, items := db.Table("user").Slice(0, 20, base.Map{
  "$sort": base.Map{"id": base.DESC},
})
if db.Error() != nil {
  // handle
}

_ = users
_ = total
_ = items
```

## 超时与取消

```go
ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
defer cancel()

db := data.Base().WithContext(ctx).WithTimeout(1500 * time.Millisecond)
defer db.Close()

rows := db.Table("user").Query(base.Map{"status": "active"})
if db.Error() != nil {
  // handle timeout/cancel/error
}
_ = rows
```

## 连接池与缓存容量

```toml
[data.main]
driver = "postgresql"
url = "postgres://..."
maxOpen = 50
maxIdle = 20
maxLifetime = "30m"
maxIdleTime = "5m"

[data.main.setting]
cache = { enable = true, ttl = "10s", capacity = 5000 }
```

```go
pools := data.GetPoolStats("main")
_ = pools
```

## Parse

```go
where, params := db.Parse(base.Map{
  "status": "active",
  "age": base.Map{"$gte": 18},
})
if db.Error() != nil {
  // handle
}
_ = where
_ = params
```

## 字段命名映射（可选）

默认关闭。开启后会做 `camelCase <-> snake_case` 映射。  
详细说明见：[字段命名映射](/zh/modules/data-field-mapping)

## 事务

```go
_ = db.Tx(func(tx data.DataBase) base.Res {
  _ = tx.Table("wallet").Update(base.Map{"$inc": base.Map{"balance": -100}}, base.Map{"id": 1})
  if tx.Error() != nil { return infra.Fail.With(tx.Error().Error()) }

  _ = tx.Table("wallet").Update(base.Map{"$inc": base.Map{"balance": 100}}, base.Map{"id": 2})
  if tx.Error() != nil { return infra.Fail.With(tx.Error().Error()) }

  return infra.OK
})
```

```go
_ = db.TxReadOnly(func(tx data.DataBase) base.Res {
  _ = tx.Table("user").Query(base.Map{"status": "active"})
  if tx.Error() != nil { return infra.Fail.With(tx.Error().Error()) }
  return infra.OK
})
```

## 子页面

- [查询 DSL](/zh/modules/data-query)
- [写入 DSL](/zh/modules/data-write)
- [高级用法](/zh/modules/data-advanced)
- [命名语义](/zh/modules/data-naming)
- [字段命名映射](/zh/modules/data-field-mapping)
- [接口迁移指南](/zh/modules/data-migration)
- [结构迁移（Migrate）](/zh/modules/data-schema-migration)
- 驱动：
  - [data-postgres](/zh/drivers/data-postgres)
  - [data-mysql](/zh/drivers/data-mysql)
  - [data-sqlite](/zh/drivers/data-sqlite)
  - [data-mongodb](/zh/drivers/data-mongodb)

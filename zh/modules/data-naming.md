---
outline: deep
---

# data 命名语义

`data` 模块采用 SQL 优先的命名：

- `Insert`: 新增单条
- `InsertMany`: 批量新增
- `Upsert`: 存在即更新，不存在则新增
- `UpsertMany`: 批量 upsert
- `Update`: 按条件更新并返回命中的第一条
- `UpdateMany`: 按条件更新全部命中
- `Delete`: 按条件删除并返回命中的第一条
- `DeleteMany`: 按条件删除全部命中

## 推荐使用

```go
item := db.Table("user").Insert(base.Map{
  "name": "alice",
})

_ = db.Table("user").Update(base.Map{
  "$set": base.Map{"status": "active"},
}, base.Map{"id": item["id"]})

_ = db.Table("user").Delete(base.Map{"id": item["id"]})
```

## Watcher 对应关系

Watcher 使用同一组写操作名：

- `Insert`
- `Update`
- `Upsert`
- `Delete`
- `Action`（监听全部写操作）

```go
infra.Register("user", data.Watcher{
  Action: func(m data.Mutation) {},
  Insert: func(m data.Mutation) {},
  Update: func(m data.Mutation) {},
  Upsert: func(m data.Mutation) {},
  Delete: func(m data.Mutation) {},
})
```

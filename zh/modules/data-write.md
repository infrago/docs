---
outline: deep
---

# data 写入 DSL

## 更新操作符

- `$set`
- `$inc`
- `$unset`
- `$push`
- `$pull`
- `$addToSet`
- `$setPath`
- `$unsetPath`

## 示例

```go
old := db.Table("user").First(base.Map{"id": 1001})
if db.Error() != nil {
  return
}

changed := db.Table("user").Update(base.Map{
  "$set": base.Map{"status": "active"},
  "$inc": base.Map{"version": 1},
  "$addToSet": base.Map{"tags": []string{"go", "backend"}},
  "$setPath": base.Map{"profile.nickname": "alice"},
}, base.Map{"id": old["id"]})
if db.Error() != nil {
  // handle
}
_ = changed
```

单条与批量更新/删除：

```go
// Update：更新并返回命中的第一条（未传 $sort 时按主键升序）
one := db.Table("user").Update(base.Map{
  "$set": base.Map{"status": "active"},
}, base.Map{"status": "pending"})

// UpdateMany：更新全部命中
many := db.Table("user").UpdateMany(base.Map{
  "$set": base.Map{"status": "active"},
}, base.Map{"status": "pending"})

// Delete：删除并返回命中的第一条
delOne := db.Table("user").Delete(base.Map{"status": "inactive"})

// DeleteMany：删除全部命中
delMany := db.Table("user").DeleteMany(base.Map{"status": "inactive"})
_, _, _, _ = one, many, delOne, delMany
```

## 批量

```go
items := db.Table("user").InsertMany([]base.Map{
  {"name": "u1"},
  {"name": "u2"},
})
if db.Error() != nil {
  // handle
}
_ = items
```

## 迁移

```go
db.Migrate("user", "order")
if db.Error() != nil {
  // handle
}
```

也可以用快捷函数：

```go
if err := data.Migrate("user", "order"); err != nil {
  // handle
}
```

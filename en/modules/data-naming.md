---
outline: deep
---

# data Naming Semantics

`data` uses SQL-first naming:

- `Insert`: insert one row/document
- `InsertMany`: insert multiple rows/documents
- `Upsert`: update if exists, otherwise insert
- `UpsertMany`: batch upsert
- `Update`: update and return first matched row/document
- `UpdateMany`: update all matched rows/documents
- `Delete`: delete and return first matched row/document
- `DeleteMany`: delete all matched rows/documents

## Recommended Usage

```go
item := db.Table("user").Insert(Map{
  "name": "alice",
})

_ = db.Table("user").Update(Map{
  "$set": Map{"status": "active"},
}, Map{"id": item["id"]})

_ = db.Table("user").Delete(Map{"id": item["id"]})
```

## Watcher Mapping

Watcher uses the same write-op names:

- `Insert`
- `Update`
- `Upsert`
- `Delete`
- `Action` (all write operations)

```go
infra.Register("user", data.Watcher{
  Action: func(m data.Mutation) {},
  Insert: func(m data.Mutation) {},
  Update: func(m data.Mutation) {},
  Upsert: func(m data.Mutation) {},
  Delete: func(m data.Mutation) {},
})
```

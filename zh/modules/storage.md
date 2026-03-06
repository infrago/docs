---
outline: deep
---

# storage

## 职责

统一文件上传、拉取、下载、删除、浏览能力，支持多存储后端。

## 配置结构

```toml
[file]
download = "store/download"
thumbnail = "store/thumbnail"
preview = "store/preview"
salt = "infrago"

[storage.default]
driver = "s3"
weight = 1
prefix = "prod"
proxy = false
remote = true

[storage.default.setting]
region = "us-east-1"
bucket = "my-bucket"
```

字段：

- `storage.<name>.driver`
- `weight` `prefix`
- `proxy` `remote`
- `setting`
- 全局 `file`：`download/thumbnail(thumb)/preview/salt`

## 对外 API

- `storage.Upload / UploadTo`
- `storage.Fetch`
- `storage.Download`
- `storage.Remove`
- `storage.Browse`
- `storage.Thumbnail / Preview`
- `storage.Decode(code)`

## 缩图处理器

- 内置图片缩略图处理器，支持 `jpg/jpeg/png/bmp/gif/webp`
- 可通过 `infra.Register("ext", storage.Thumbnailer{...})` 注册自定义缩图处理器
- 可通过 `infra.Register("ext", storage.Previewer{...})` 注册预览处理器

## 驱动

- `default`（本地文件系统）
- [storage-minio](/zh/drivers/storage-minio)
- [storage-s3](/zh/drivers/storage-s3)

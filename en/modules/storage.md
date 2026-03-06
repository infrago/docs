# storage

Unified file/object storage module.

API:

- `Upload`, `Fetch`, `Download`, `Remove`, `Browse`, `Thumbnail`, `Preview`
- `Decode(code)`

Handlers:

- Built-in image thumbnail handler supports `jpg/jpeg/png/bmp/gif/webp`
- Register custom thumbnail handlers with `infra.Register("ext", storage.Thumbnailer{...})`
- Register preview handlers with `infra.Register("ext", storage.Previewer{...})`

Drivers:

- `default`
- `storage-minio`
- `storage-s3`

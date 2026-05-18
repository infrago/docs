# config-file

Reads local configuration files into a runtime `Map`.

## Driver

- `file`

## Parameters

- `file` / `path` / `config`: configuration file path
- `format`: `toml`, `json`, or `yaml`; when omitted, the driver detects by extension/content

## Example

```bash
./app --driver=file --file=config.toml
```

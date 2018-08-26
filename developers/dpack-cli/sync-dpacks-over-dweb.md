# Sync dPacks over dWeb

## Usage

```
dweb sync [<dir>] [--no-import] [--no-watch]
```

- Start distributing your dPack vault over the network.
- `dweb` will import new or updated files to the dWeb since you last ran `create` or `sync`.
- `dweb` watches files for changes and imports updated files.

* Use `--no-import` to not import any new or updated files to the dWeb.
* Use `--no-watch` to not watch the dPack directory for changes. `--import` must be true for `--watch` to work.

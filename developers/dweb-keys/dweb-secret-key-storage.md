# dWeb Secret Key Storage

## Install

```
npm install @dpack/secret-storage
```

## Usage

Return for the `secret_key` storage in dDrive/dDatabase. To avoid local ownership conflicts, pass the local directory as the first argument. `@dpack/secret-storage` will check for a non-empty ownership file in the source directory storage.

```js
var secretStore = require('@dpack/secret-storage')

var storage = {
  metadata: function (name, opts) {
    if (name === 'secret_key') return secretStore()(path.join(dir, '.dweb/metadata.ogd'), opts)
    return // other storage
  },
  content: function (name, opts) {
    return // other storage
  }
}

// store secret key in ~/.dweb/secret_keys
var vault = ddrive(storage)
```

## API

### `secretStorage([dir])(ownershipFile, opts)`

* `dir`: directory to store keys under `dir/.dweb/secret_keys`. Defaults to users home directory.
* `ownershipFile`: non-empty file that denotes ownership. This helps avoid local ownership conflicts of the same dPack.

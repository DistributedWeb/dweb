# dDrive Transfer

## Introduction
Import the contents of a folder into a [dDrive](https://ddrive.io), and optionally keep watching for changes.

## Example

```js
const ddrive = require('@ddrive/core')
const memdb = require('memdb')
const dDriveImport = require('@ddrive/transfer')

const drive = ddrive(memdb())
const vault = drive.createVault()

dDriveImport(vault, 'a/directory/', err => {
  // ...
})
```

## Installation

```bash
$ npm install @ddrive/transfer
```

## API

### dDriveImport(vault, target, [, options][, cb])

Recursively import `target`, which is the path to a directory or file,  into `vault` and call `cb` with the potential error. The import happens sequentually. Returns a `status` object.

Options

- `live`: keep watching
- `resume`: assume the vault isn't fresh
- `basePath`: where in the vault should the files import to? (defaults to '')
- `ignore`: [anymatch](https://npmjs.org/package/anymatch) expression to ignore files

To enable watching, set `live: true`, like this:

```js
const status = dDriveImport(vault, target, { live: true }, err => {
  console.log('initial import done')  
})
status.on('error', err => {
  // ...  
})
// when you want to quit:
status.close()
```

If you want to resume importing an already existing vault, set `resume: true `. This module then checks a file's size and mtime to determine whether it needs to be updated or created.

If you want to import into a subfolder, set `basePath`:

```js
dDriveImport(vault, target, { basePath: '/some/subdir' }, err => {...})
```

### status

Events:

- `error` (`err`)
- `file imported` ({ `path`, `mode=updated|created` })
- `file skipped` ({ `path` })

Properties:

- `fileCount`: The count of currently known files
- `totalSize`: Total file size in bytes

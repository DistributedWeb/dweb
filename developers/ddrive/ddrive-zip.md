# dDrive ZIP

## Introduction
Library For Create A Zip File From a dDrive.

## Usage

```js
const toZipStream = require('@ddrive/zip')

toZipStream(vault).pipe(fs.createWriteStream(...))
```

The output zip will only contain files that are fully downloaded.

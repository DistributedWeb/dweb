# dDrive Fork

## Introduction
Check if a file on the filesystem is the same as an entry in dDrive by comparing `stat` objects and file contents.

* Compare `vault.stat` and `fs.stat` first, then
* Compare `vault.createReadStream` and `fs.createReadStream` (will fail as soon as stream differs).

**Warning! Can be decrease performance to check duplicates of many large files.**

## Usage

```js
var isFork = require('@ddrive/fork')

var vault = ddrive(dwRem)

vault.writeFile('example.js', fs.readFileSync('example.js'), function (err) {
  if (err) throw err
  // example.js is now in the vault
  // we can see if the fs file is duplicate
  isFork(vault, 'example.js', function (err, duplicate) {
    if (err) throw err
    if (duplicate) console.log('example.js is duplicate!')
  })

  isFork(vault, 'index.js', 'example.js', function (err, duplicate) {
    // index.js is a file on our fs
    // example.js is the file in our vault
    if (err) throw err
    if (duplicate) console.log('index.js not a duplicate of example.js!')
  })
})
```

## API

### isFork(vault, filePath, [entryName], cb)

Callback returns `(err, isFork)` where `isFork` is a boolean, true if the file is a duplicate.

If `filePath` is different from the entry name in dDrive, specify both.

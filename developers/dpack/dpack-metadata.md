# dPack Metadata

## Introduction

Read and Write `dweb.json` Files

## Install

### Install With NPM
```
npm install @dpack/metadata
```

### Install With YARN
```
yarn add @dpack/metadata
```

## Usage

```js
var dWebMeta = require('@dpack/metadata')

var dwebjson = dWebMeta(vault)

dwebjson.create({title: 'a dPack', description: 'this is my dPack description'}, function (err) {
  if (err) throw err
})

dwebjson.read(function (err, data) {
  console.log(data)
})
```

Write to a `dweb.json` on the file system also:

```js
var dWebMeta = require('@dpack/metadata')

var dwebjson = dWebMeta(vault, {file: path.join(dweb.path, 'dweb.json')})

dwebjson.create({title: 'dPack title', description: 'This is my awesome dSite.'}, function (err) {
  if (err) throw err
})
```

## API

### `var dwebjson = dWebMeta(vault, [opts])`

creates a new dwebMeta db

Options:

* `opts.file` - dweb.json file path, updates will be written to file system and vault

#### `dwebjson.create([data], cb)`

Create a new `dweb.json` file in the vault with the default keys (`url`, `title`, `description`). Pass in any additional data to add on initial create.

#### `dwebjson.write(key, val, cb)` or `dwebjson.write(data, cb)`

Write a single `key` and `value` or an object, `data`, to the `dweb.json` file. Use `file` option above to also update the file on the file system.

#### `dwebjson.delete(key, cb)`

Delete a `key` from the `dweb.json` file.

#### `dwebjson.read(cb)`

Read the current `dweb.json`.

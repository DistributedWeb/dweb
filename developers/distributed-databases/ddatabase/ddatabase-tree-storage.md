# dDatabase Tree Storage

## Introduction
A module that stores dDatabase metadata trees using a storage provider.

```
npm install @ddatabase/tree-storage
```

## Usage

``` js
var treeStorage = require('@ddatabase/tree-storage')
var storage = require('@dwcore/ref')

var tree = treeStorage(storage('test.tree'))

tree.put(0, {
  hash: crypto.createHash('sha256').update('test').digest(),
  size: 4
}, function (err) {
  if (err) throw err
  tree.get(0, console.log)
})
```

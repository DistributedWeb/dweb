# Multi dDrive Manager

## Introduction 
Manage multiple dDrive vaults located anywhere on the filesystem.

## Installation
```sh
$ npm install @ddrive/multi
```

## Usage
```js
var ddrive = require('@ddrive/core')
var multiDDrive = require('@ddrive/multi')
var toilet = require('toiletdb')

var store = toilet('./data.json')
multiDDrive(store, createVault, closeVault, function (err, dDrive) {
  if (err) throw err

  var data = { key: '<64-bit-hex>' }
  dDrive.create(data, function (err, vault) {
    if (err) throw err

    var vaults = dDrive.list()
    console.log(vaults)

    dDrive.close(vault.key, function (err) {
      if (err) throw err
      console.log('vault deleted')
    })
  })
})

function createVault (data, done) {
  var db = level('/tmp/' + 'multiDDrive-' + data.key)
  var dDrive = ddrive(db)
  var vault = dDrive.createVault(data.key)
  done(null, vault)
}

function closeVault (vault, done) {
  vault.close()
  done()
}
```

## Error handling
If there is an error initializing a dDrive, instead of the whole process failing, an error object with attached `.data` property will be pushed into the list of vaults instead. That means when consuming multiDDrive.list(), you should check for errors:

```js
var vaults = multiDDrive.list()
vaults.forEach(function (vault) {
  if (vault instanceof Error) {
    var err = vault
    console.log('failed to initialize vault with %j: %s', err.data, err.message)
  }
})
```

This way you can decide for yourself whether an individual initialization failure should cause the whole process to fail or not.

## API
### multiDDrive(store, createVault, closeVault, callback(err, dDrive))
Create a new multiDDrive instance. `db` should be a valid `toiletdb` instance.
`createVault` is the function used to create new dDrive vaults.
`callback` is called after initialization. `closeVault` is called when
`dDrive.remove()` is called.

`createVault` has an api of `createVault(data, done)` where `data` is passed in
by `dDrive.create()` and `done(err, vault)` expects a valid vault.

`closeVault` has an api of `closeVault(vault, done)` where `vault` was
created by `createVault` and `done(err)` is expected to be called when the
vault has been properly closed. `closeVault` is called when a specific
vault is closed through `.close` or when through `.disconnect` all vaults get
disconnected.

### vaults = dDrive.list()
List all `vaults` in the `multiDDrive`.

### dDrive.create(data, callback(err, dDrive[, duplicate]))
Create a new dDrive vault. `data` is passed into `createVault`.
If an vault with the same key already exists, returns that instead and sets
`duplicate` to `true`.

### dDrive.close(key, callback(err))
Remove an vault by its public key. Calls `closeVault()`

### dDrive.disconnect(callback(err))
Disconnects the dDrive from the store and closes all vaults (without removing them).

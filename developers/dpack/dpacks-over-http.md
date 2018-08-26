# dPacks Over HTTP

## Introduction 
An HTTP transport/storage provider for dPacks, allowing replication of dPacks over normal HTTP connections from flat files on the server. Currently only supports read operations, write operations coming in the future (open an issue if you need this).

The entire `.dweb` folder must be available on the server for this to work. Point this at the root url where the `.dweb` folder is and you can use this to do replication.

This is implemented as a storage provider, conforming to the https://www.npmjs.com/package/abstract-random-access API. That may seem counterintuitive, as this provides a networking transport but implements a storage provider API. However, with dPack you can wrap a storage provider in a dDrive instance to turn it into a network transport.

### example

```js
var ddrive = require('@ddrive/core')
var DWeb = require('@dpack/core')
var dPackHttp = require('@dpack/http')

// httpDDrive is a drive that 'wraps' the http storage provider in a dDrive instance. It won't write any data to disk, in fact we won't be using it to write anything, only to read things.
var storage = dPackHttp('https://dpack-location.com')
var httpDDrive = ddrive(storage, {latest: true})

httpDDrive.on('ready', function () {
  // create a 'real' dPack using the default storage in a local folder
  DWeb('./local-copy', {key: httpDDrive.key, thin: true}, function (err, dweb) {
    if (err) throw err
    // heres the magic that hooks up the http read only dDrive to our real one
    var localReplicate = dweb.vault.replicate()
    var httpReplicate = httpDDrive.replicate()
    localReplicate.pipe(httpReplicate).pipe(localReplicate)

    // after this point, the 'real' dPack will use the http dPack as a source for all data
  })
})
```

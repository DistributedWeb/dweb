# dDrive HTTP

## Introduction
Serve a [ddrive](https://ddrive.io) vault or [dDatabase](https://ddatabase.io) `Ddb` over HTTP.

## Usage

`ddrive-http` returns a function to call when you receive a http request:

```js
var server = http.createServer()
server.on('request', ddriveHttp(vault))
```

### Setup

To use `ddrive-http` you will need to:
* Create your own http server
* Setup your dDrive vault(s)
* Connect to the flock before serving the dDrive vault

### API

dDrive works with many vaults/Ddbs or a single vault.

#### Multiple Vaults

If you have multiple vaults, you will need to look the vault to return using the key.

Initiate with an vault lookup function:
`var onrequest = ddriveHttp(getVault)`

The vault lookup function may look like this:

```js
var getVault = function (dwebInfo, cb) {
  // dwebInfo = {
  //   key: vault.key,
  //   filename: filename.txt // If file is requested in URL
  //   op: 'get' or 'changes'
  // }

  // Find the vault to return:
  var vault = cache.get(dwebInfo.key)
  if (!vault) {
    vault = drive.createVault(dwebInfo.key)
    // Make sure you join the flock before callback
    sw.join(vault.revelationKey)
  }
  cb(null, vault) // callback with your found vault
}
```

#### Single Vault
dDrive-http works great with a single vault too. It exposes the metadata at the root path and files are available without using the key.

Pass a single vault on initiation:
`var onrequest = ddriveHttp(vault)`

Now your vault metadata will be available at http://example.com/

#### dDatabase Ddb(s)
You can also use a dDatabase ddb: `ddriveHttp(ddb)` (or using a similar getVault function)

### URL Format

`ddrive-http` responds to any URL with a specific format. If the URL does cannot be parsed, it will return a 404.

#### Multiple vaults on one site

* Get metadata for vault: `http://dwebs.io/c5dbfe5521d8dddba683544ee4b1c7f6ce1c7b23bd387bd850397e4aaf9afbd9/`
* Get file from vault: `http://dwebs.io/c5dbfe5521d8dddba683544ee4b1c7f6ce1c7b23bd387bd850397e4aaf9afbd9/filename.pdf`
* Upload file: `POST http://example-vault.dwebs.io/` or `POST http://example-vault.dwebs.io/c5dbfe5521d8dddba683544ee4b1c7f6ce1c7b23bd387bd850397e4aaf9afbd9`

#### Single Vault Mode

* Get metadata for vault: `http://example-vault.dwebs.io/`
* Get file from vault: `http://example-vault.dwebs.io/filename.pdf`
* Upload file: `POST http://example-vault.dwebs.io/`

#### dDatabase Mode

For dDatabase Ddbs, the data is available with the same logic as above for a single or multiple Ddbs.


## Example

```javascript
var ddriveHttp = require('@ddrive/http')

var getVault = function (dwebInfo, cb) {
  // find the vault to serve
  var revelationKey = crypto.createHmac('sha256', Buffer(dwebInfo.key, 'hex')).update('ddatabase').digest('hex')
  var vault = cache.get(revelationKey)
  if (!vault) {
    vault = drive.createVault(dwebInfo.key)
    // connect to flock, if necessary
    sw.join(vault.revelationKey)
  }
  cb(null, vault) // callback with your found vault
}

var onrequest = ddriveHttp(getVault)
var server = http.createServer()
server.listen(8000)
server.on('request', onrequest)
```

Pass an vault lookup function for the first argument of `ddriveHttp`. The function is called with `dwebInfo` and a callback.

```javascript
dwebInfo = {
  key: vault.key,
  filename: someFile.txt,
  op: 'get' // or 'changes'
}
```

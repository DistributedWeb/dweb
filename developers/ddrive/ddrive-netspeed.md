# dDrive NetSpeed

## Introduction
Get upload and download speeds for a dDrive Vault

## Usage

```js
var vault = ddrive('.dweb')
var flock = flockRevelation(vault)
var dwnetspeed = dWebNetSpeed(vault, {timeout: 1000})

setInterval(function () {
  console.log('upload speed: ', dwnetspeed.uploadSpeed)
  console.log('download speed: ', dwnetspeed.downloadSpeed)
}, 500)
```

## API

### `var speed = networkSpeed(vault, [opts])`

* `vault` is a dDrive vault.
* `opts.timeout` is the only option. Speed will be reset to zero after the timeout.

#### `dwnetspeed.uploadSpeed`

Vault upload speed across all peers.

#### `dwnetspeed.downloadSpeed`

Vault download speed across all peers.

#### `dwnetspeed.downloadTotal`

Vault total download.

#### `dwnetspeed.uploadTotal`

Vault total upload.

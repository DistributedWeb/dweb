# dWeb's Flock Core Library

## Introduction

Join the dWeb P2P flock for [dDatabase](/developer-tools/distributed-databases/ddatabase) and [dDrive](/developer-tools/distributed-databases/ddrive) distributed feeds. Uses [@flockcore/revelation](/developer-tools/flockcore/flock-revelation) as a dependency.

```
npm install @flockcore/core
```

## Usage

Run the following code in two different places and they will replicate the contents of the given `VAULT_KEY`.

```js
var ddrive = require('@ddrive/core')
var flockCore = require('@flockcore/core')

var vault = ddrive('./ddatabase', 'VAULT_KEY')
var flock = flockCore(vault)
flock.on('connection', function (peer, type) {
  console.log('got', peer, type)
  console.log('connected to', flock.connections.length, 'peers')
  peer.on('close', function () {
    console.log('peer disconnected')
  })
})
```

Will use `@flockcore/revelation` to attempt to connect peers. Uses `@flockcore/presets` for peer introduction presets on the server side, which can be overwritten (see below).

The module can also create and join a flock for a dDatabase distributed feed:

```js
var ddatabase = require('@ddatabase/core')
var flockCore = require('@flockcore/core')

var ddb = ddatabase('/ddb')
var flock = flockCore(ddb)
```

## API

### `var flock = flockCore(vault, opts)`

Join the p2p flock for the given distributed feed. The return object, `flock`, is an event emitter that will emit a `peer` event with the peer information when a peer is found.

### flock.connections

Get the list of currently active connections.

### flock.close()

Exit the flock

##### Options

  * `stream`: function, replication stream for connection. Default is `vault.replicate({live, upload, download})`.
  * `upload`: bool, upload data to the other peer?
  * `download`: bool, download data from the other peer?
  * `port`: port for flock revelation
  * `utp`: use utp in flock revelation
  * `tcp`: use tcp in flock revelation

Defaults from `@flockcore/presets` can also be overwritten:

  * `dns.server`: DNS server
  * `dns.domain`: DNS domain
  * `dht.bootstrap`: distributed hash table bootstrapping nodes

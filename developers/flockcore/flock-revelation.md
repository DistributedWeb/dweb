# dWeb's Flock Revelation

## Introduction

dWeb's network flock that uses [@flockcore/channel](http://open.benchx.io/flockcore/channel) to find and connect to other dWeb peers.

This module implements peer connection state and builds on revelation-channel which implements peer revelation. This uses TCP sockets by default and has experimental support for UTP.

```
npm install @flockcore/revelation
```

## Usage

``` js
var flockRevelation = require('@flockcore/revelation')

var flock = flockRevelation()

flock.listen(1000)
flock.join('ubuntu-16.04') // can be any id/name/hash

flock.on('connection', function (connection) {
  console.log('found + connected to dWeb peer')
})
```

## API

#### `var flock = flockRevelation(opts)`

Create a new flock. Options include:

```js
{
  id: crypto.randomBytes(32), // dWeb peer-id for user
  stream: stream, // stream to replicate across dWeb peers
  connect: fn, // connect local and remote streams yourself
  utp: true, // use utp for revelation
  tcp: true, // use tcp for revelation
  maxConnections: 0, // max number of connections.
  whitelist: [] // array of ip addresses to restrict connections to
}
```

For full list of `opts` take a look at [@flockcore/channel](http://open.benchx.io/flockcore/channel)

#### `flock.join(key, [opts], [cb])`

Join a channel specified by `key` (usually a name, hash or id, must be a **Buffer** or a **string**). After joining will immediately search for peers advertising this key, and re-announce on a timer.

If you pass `opts.announce` as a falsy value you don't announce your port (e.g. you will be in discover-only mode)

If you specify cb, it will be called *when the first round* of revelation has completed. But only on the first round.

#### `flock.leave(key)`

Leave the channel specified `key`

#### `flock.connecting`

Number of peers we are trying to connect to

#### `flock.queued`

Number of peers revelated but not connected to yet

#### `flock.connected`

Number of peers connected to

#### `flock.on('peer', function(peer) { ... })`

Emitted when a peer has been revelated. Peer is an object that contains info about the peer.

```js
{
  channel: Buffer('...'), // the channel this connection was initiated on.
  host: '127.0.0.1', // the remote address of the peer.
  port: 8080, // the remote port of the peer.
  id: '192.168.0.5:64374@74657374', // the remote peer's peer-id.
  retries: 0 // the number of times tried to connect to this peer.
}
```


#### `flock.on('peer-banned', function(peerAddress, details) { ... })`

Emitted when a peer has been banned as a connection candidate. peerAddress is an object that contains info about the peer. Details is an object that describes the rejection.

```js
{
  reason: 'detected-self' // may be 'application' (removePeer() was called) or 'detected-self'
}
```

#### `flock.on('peer-rejected', function(peerAddress, details) { ... })`

Emitted when a peer has been rejected as a connection candidate. peerAddress is an object that contains info about the peer. Details is an object that describes the rejection

```js
{
  reason: 'duplicate' // may be 'duplicate', 'banned', or 'whitelist'
}
```

#### `flock.on('drop', function(peer) { ... })`

Emitted when a peer has been dropped from tracking, typically because it failed too many times to connect. Peer is an object that contains info about the peer.

#### `flock.on('connecting', function(peer) { ... })`

Emitted when a connection is being attempted. Peer is an object that contains info about the peer.

#### `flock.on('connect-failed', function(peer, details) { ... })`

Emitted when a connection attempt fails. Peer is an object that contains info about the peer. Details is an object that describes the failure

```js
{
  timedout: true // was the failure a timeout?
}
```

#### `flock.on('handshaking', function(connection, info) { ... })`

Emitted when you've connected to a peer and are now initializing the connection's session. Info is an object that contains info about the connection.

``` js
{
  type: 'tcp', // the type, tcp or utp.
  initiator: true, // whether we initiated the connection or someone else did.
  channel: Buffer('...'), // the channel this connection was initiated on. only set if initiator === true.
  host: '127.0.0.1', // the remote address of the peer.
  port: 8080, // the remote port of the peer.
  id: Buffer('...') // the remote peer's peer-id.
}
```

#### `flock.on('handshake-timeout', function(connection, info) { ... })`

Emitted when the handshake fails to complete in a timely fashion. Info is an object that contains info about the connection.

#### `flock.on('connection', function(connection, info) { ... })`

Emitted when you have fully connected to another peer. Info is an object that contains info about the connection.

#### `flock.on('connection-closed', function(connection, info) { ... })`

Emitted when you've disconnected from a peer. Info is an object that contains info about the connection.

#### `flock.on('redundant-connection', function(connection, info) { ... })`

Emitted when multiple connections are detected with a peer, and so one is going to be dropped (the `connection` given). Info is an object that contains info about the connection.

#### `flock.listen(port)`

Listen on a specific port. Should be called before add

dWeb Peers Javascript Library
---

# @dwebs/peer

## Features

- concise, **node.js style** API for [WebRTC](https://wiki.dwebs.io/WebRTC)
- **works in node and the browser!**
- supports **video/voice streams**
- supports **data channel**
  - text and binary data
  - node.js [duplex stream](http://nodejs.org/api/stream.html) interface
- supports advanced options like:
  - enable/disable [trickle ICE candidates](http://webrtchacks.com/trickle-ice/)
  - manually set config and constraints options

This module works in the browser with [browserify](http://browserify.org/).

**Note:** If you're **NOT** using browserify, then use the included standalone file
`dwebpeer.min.js`. This exports a `DWebPeer` constructor on `window`.

## Installation

```
npm install @dwebs/peer
```

## usage

These examples create two peers in the same page.

In a real-world application, the sender and receiver `DWebPeer` instances would exist in
separate browsers. A "beaming server" (usually implemented with websockets) would be
used to exchange beaming data between the two browsers until a peer-to-peer connection
is established.

### data channels

```js
var DWebPeer = require('@dwebs/peer')

var dWebPeer1 = new DWebPeer({ initiator: true })
var dWebPeer2 = new DWebPeer()

dWebPeer1.on('beam', function (data) {
  // when dWebPeer1 has beaming data, give it to dWebPeer2
  dWebPeer2.beam(data)
})

dWebPeer2.on('beam', function (data) {
  // same as above, but in reverse
  dWebPeer1.beam(data)
})

dWebPeer1.on('connect', function () {
  // wait for 'connect' event before using the data channel
  dWebPeer1.send('hey dWebPeer2, how is it going?')
})

dWebPeer2.on('data', function (data) {
  // got a data channel message
  console.log('got a message from dWebPeer1: ' + data)
})
```

### video/voice

Video/voice is also super simple! In this example, dWebPeer1 sends video to dWebPeer2.

```js
var DWebPeer = require('@dwebs/peer')

// get video/voice stream
navigator.getUserMedia({ video: true, audio: true }, gotMedia, function () {})

function gotMedia (stream) {
  var dWebPeer1 = new DWebPeer({ initiator: true, stream: stream })
  var dWebPeer2 = new DWebPeer()

  dWebPeer1.on('beam', function (data) {
    dWebPeer2.beam(data)
  })

  dWebPeer2.on('beam', function (data) {
    dWebPeer1.beam(data)
  })

  dWebPeer2.on('stream', function (stream) {
    // got remote video stream, now let's show it in a video tag
    var video = document.querySelector('video')
    video.src = window.URL.createObjectURL(stream)
    video.play()
  })
}
```

For two-way video, simply pass a `stream` option into both `Peer` constructors. Simple!

## real-world apps that use `dweb-peer`

- [Instant](https://instant.io) - Secure, anonymous, streaming file transfer
- [WebTorrent](http://webtorrent.io) - Streaming torrent client in the browser
- [PusherTC](http://pushertc.herokuapp.com) - Video chat with using Pusher. See [guide](http://blog.carbonfive.com/2014/10/16/webrtc-made-simple/).
- [lxjs-chat](https://github.com/feross/lxjs-chat) - Omegle-like video chat site
- *Your app here! - send a PR!*

## api

### `peer = new DWebPeer([opts])`

Create a new WebRTC peer connection.

A "data channel" for text/binary communication is always established, because it's cheap and often useful. For video/voice communication, pass the `stream` option.

If `opts` is specified, then the default options (shown below) will be overridden.

```
{
  initiator: false,
  stream: false,
  config: { iceServers: [ { url: 'stun:23.21.150.121' } ] },
  constraints: {},
  channelName: '<random string>',
  trickle: true
}
```

The options do the following:

- `initiator` - set to true if this is the initiating peer
- `stream` - if video/voice is desired, pass stream returned from `getUserMedia`
- `config` - custom webrtc configuration
- `constraints` - custom webrtc video/voice constaints
- `channelName` - custom webrtc data channel name
- `trickle` - set to `false` to disable [trickle ICE](http://webrtchacks.com/trickle-ice/) and get a single 'beam' event (slower)

### `peer.beam(data)`

Call this method whenever the remote peer emits a `peer.on('beam')` event.

The `data` will be a `String` that encapsulates a webrtc offer, answer, or ice candidate. These messages help the peers to eventually establish a direct connection to each other. The contents of these strings are an implementation detail that can be ignored by the user of this module; simply pass the data from 'beam' events to the remote peer, call `peer.beam(data)`, and everything will just work.

### `peer.send(data)`

Send text/binary data to the remote peer. `data` can be any of several types: `String`, `Buffer` (see [buffer](https://github.com/feross/buffer)), `TypedArrayView` (`Uint8Array`, etc.), `ArrayBuffer`, or `Blob` (in browsers that support it).

Other data types will be transformed with `JSON.stringify` before sending. This is handy
for sending object literals across like this:
`peer.send({ type: 'data', data: 'hi' })`.

Note: If this method is called before the `peer.on('connect')` event has fired, then data
will be buffered.

### `peer.destroy([onclose])`

Destroy and cleanup this peer connection.

If the optional `onclose` parameter is passed, then it will be registered as a listener on the 'close' event.

### duplex stream

`Peer` objects are instances of `stream.Duplex`. The behave very similarly to a
`net.Socket` from the node core `net` module. The duplex stream reads/writes to the data
channel.

```js
var dWebPeer = new DWebPeer(opts)
// ... beaming ...
dWebPeer.write(new Buffer('hey'))
dWebPeer.on('data', function (chunk) {
  console.log('got a chunk', chunk)
})
```

## events


### `peer.on('beam', function (data) {})`

Fired when the peer wants to send beaming data to the remote peer.

**It is the responsibility of the application developer (that's you!) to get this data to the other peer.** This usually entails using a websocket beaming server. Then, simply call `peer.beam(data)` on the remote peer.

### `peer.on('connect', function () {})`

Fired when the peer connection and data channel are ready to use.

### `peer.on('data', function (data) {})`

Received a message from the remote peer (via the data channel).

`data` will be either a `String` or a `Buffer/Uint8Array` (see [buffer](https://github.com/feross/buffer)).

### `peer.on('stream', function (stream) {})`

Received a remote video stream, which can be displayed in a video tag:

```js
peer.on('stream', function (stream) {
  var video = document.createElement('video')
  video.src = window.URL.createObjectURL(stream)
  document.body.appendChild(video)
  video.play()
})
```

### `peer.on('close', function () {})`

Called when the peer connection has closed.

### `peer.on('error', function (err) {})`

Fired when a fatal error occurs. Usually, this means bad beaming data was received from the remote peer.

`err` is an `Error` object.

## connecting more than 2 peers?

The simplest way to do that is to create a full-mesh topology. That means that every peer
opens a connection to every other peer. To illustrate:

![full mesh topology](img/full-mesh.png)

To broadcast a message, just iterate over all the peers and call `peer.send`.

So, say you have 3 peers. Then, when a peer wants to send some data it must send it 2
times, once to each of the other peers. So you're going to want to be a bit careful about
the size of the data you send.

Full mesh topologies don't scale well when the number of peers is very large. The total
number of edges in the network will be ![full mesh formula](img/full-mesh-formula.png)
where `n` is the number of peers.

For clarity, here is the code to connect 3 peers together:

#### Peer 1

```js
// These are dWebPeer1's connections to dWebPeer2 and dWebPeer3
var dWebPeer2 = new DWebPeer({ initiator: true })
var dWebPeer3 = new DWebPeer({ initiator: true })

dWebPeer2.on('beam', function (data) {
  // send this beaming data to dWebPeer2 somehow
})

dWebPeer2.on('connect', function () {
  dWebPeer2.send('hi dWebPeer2, this is dWebPeer1')
})

dWebPeer2.on('data', function (data) {
  console.log('got a message from dWebPeer2: ' + data)
})

dWebPeer3.on('beam', function (data) {
  // send this beaming data to dWebPeer3 somehow
})

dWebPeer3.on('connect', function () {
  dWebPeer3.send('hi dWebPeer3, this is dWebPeer1')
})

dWebPeer3.on('data', function (data) {
  console.log('got a message from dWebPeer3: ' + data)
})
```

#### Peer 2

```js
// These are dWebPeer2's connections to dWebPeer1 and dWebPeer3
var dWebPeer1 = new DWebPeer()
var dWebPeer3 = new DWebPeer({ initiator: true })

dWebPeer1.on('beam', function (data) {
  // send this beaming data to dWebPeer1 somehow
})

dWebPeer1.on('connect', function () {
  dWebPeer1.send('hi dWebPeer1, this is dWebPeer2')
})

dWebPeer1.on('data', function (data) {
  console.log('got a message from dWebPeer1: ' + data)
})

dWebPeer3.on('beam', function (data) {
  // send this beaming data to dWebPeer3 somehow
})

dWebPeer3.on('connect', function () {
  dWebPeer3.send('hi dWebPeer3, this is dWebPeer2')
})

dWebPeer3.on('data', function (data) {
  console.log('got a message from dWebPeer3: ' + data)
})
```

#### Peer 3

```js
// These are dWebPeer3's connections to dWebPeer1 and dWebPeer2
var dWebPeer1 = new DWebPeer()
var dWebPeer2 = new DWebPeer()

dWebPeer1.on('beam', function (data) {
  // send this beaming data to dWebPeer1 somehow
})

dWebPeer1.on('connect', function () {
  dWebPeer1.send('hi dWebPeer1, this is dWebPeer3')
})

dWebPeer1.on('data', function (data) {
  console.log('got a message from dWebPeer1: ' + data)
})

dWebPeer2.on('beam', function (data) {
  // send this beaming data to dWebPeer2 somehow
})

dWebPeer2.on('connect', function () {
  dWebPeer2.send('hi dWebPeer2, this is dWebPeer3')
})

dWebPeer2.on('data', function (data) {
  console.log('got a message from dWebPeer2: ' + data)
})
```

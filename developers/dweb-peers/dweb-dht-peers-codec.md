# dWeb/DHT Peers Codec

## Introduction

A [dWeb Compliant Codec](/standard/encoding) for encoding a list of IPV4 peers to buffers.

```
npm install @dwdht/peers
```

## Usage

``` js
var dWebCodec = require('@dwdht/peers')

var dWebBuffer = dWebCodec.encode([{
  host: '127.0.0.1',
  port: 8080
}, {
  host: '127.0.0.1',
  port: 9090
}])

console.log(dWebBuffer) // 12 byte buffer
console.log(dWebCodec.decode(buf)) // the peer list
```

## API

#### `var dWebBuffer = dWebCodec.encode(peerList, [buffer], [offset])`

Encode a list of IPV4 peers into a buffer.

#### `var dWebDecode = dWebCodec.decode(buffer, [offset], [end])`

Decode a buffer into a list of peers.

#### `var dWebLen = dWebPeersCodec.encodingLength(peerList)`

Returns the amount of bytes needed to encode the peers into a buffer

#### `var dWebIdLen = dWebPeersCodec.idLength(idByteLength)`

Create a new `@dwdht/peers` decoder that encodes/decodes a fixed size `peerId` in addition to host/port. The `peerId` is exposed as the `.id` property on a peer object.

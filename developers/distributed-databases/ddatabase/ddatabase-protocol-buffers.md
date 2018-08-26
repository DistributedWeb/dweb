# dDatabase Protocol Buffers

## Introduction
Stream that implements the [@ddatabase/core](/core) protocol

```
npm install @ddatabase/protocol
```

## Usage

``` js
var protocol = require('@ddatabase/protocol')
var stream = protocol()

// open a ddb specified by a 32 byte key
var ddb = stream.ddb(Buffer.from('thisiswhatyouarebuffering'))

ddb.request({block: 42})
ddb.on('data', function (message) {
  console.log(message) // contains message.index and message.value
})

stream.pipe(anotherStream).pipe(stream)
```

## API

#### `var stream = protocol([options])`

Create a new protocol duplex stream.

Options include:

``` js
{
  id: optionalPeerId, // you can use this to detect if you connect to yourself
  live: keepStreamOpen, // signal to the other peer that you want to keep this stream open forever
  ack: false, // Explicitly ask a peer to acknowledge each received block
  userData: opaqueUserData // include user data that you can retrieve on handshake
  encrypt: true, // set to false to disable encryption if you are already piping through a encrypted stream
  timeout: 5000 // stream timeout. set to 0 or false to disable.
}
```

If you don't specify a peer id a random 32 byte will be used.
You can access the peer id using `p.id` and the remote peer id using `p.remoteId`.

#### `var ddb = stream.ddb(key)`

Signal the other end that you want to share a dDatabase.

You can use the same stream to share more than one BUT the first `ddb` shared
should be the same one. The key of the first `ddb` is also used to encrypt the stream using [libsodium](https://github.com/distributedweb/sodium-native#crypto_stream_xorcipher-message-nonce-key).

#### `stream.on('handshake')`

Emitted when a protocol handshake has been received. Afterwards you can check `.remoteId` to get the remote peer id, `.remoteLive` to get its live status, or `.remoteUserData` to get its user data.

#### `stream.on('ddb', revelationKey)`

Emitted when a remote is sharing a `ddb`. `revelationKey` is the dDatabase revelation key of the `ddb` they want to share.

If you are sharing multiple dDatabases on the same port you can use this event to wait for the remote peer to indicate which dDatabase
they are interested in.

#### `stream.destroy([error])`

Destroy the stream. Closes all `Ddbs` as well.

#### `stream.finalize()`

Gracefully end the stream. Closes all `Ddbs` as well.

#### `ddb.info(message)`

Send an `info` message. See the [schema.proto](schema.proto) file for more information.

#### `ddb.on('info', message)`

Emitted when an `info` message has been received.

#### `ddb.have(message)`

Send a `have` message. See the [schema.proto](schema.proto) file for more information.

#### `ddb.on('have', message)`

Emitted when a `have` message has been received.

#### `ddb.unhave(message)`

Send a `unhave` message. See the [schema.proto](schema.proto) file for more information.

#### `ddb.on('unhave', message)`

Emitted when a `unhave` message has been received.

#### `ddb.want(want)`

Send a `want` message. See the [schema.proto](schema.proto) file for more information.

#### `ddb.on('want', want)`

Emitted when a `want` message has been received.

#### `ddb.unwant(unwant)`

Send a `unwant` message. See the [schema.proto](schema.proto) file for more information.

#### `ddb.on('unwant', unwant)`

Emitted when a `unwant` message has been received.

#### `ddb.request(request)`

Send a `request` message. See the [schema.proto](schema.proto) file for more information.

#### `ddb.on('request', request)`

Emitted when a `request` message has been received.

#### `ddb.cancel(cancel)`

Send a `cancel` message. See the [schema.proto](schema.proto) file for more information.

#### `ddb.on('cancel', cancel)`

Emitted when a `cancel` message has been received.

#### `ddb.data(data)`

Send a `data` message. See the [schema.proto](schema.proto) file for more information.

#### `ddb.on('data', data)`

Emitted when a `data` message has been received.

#### `ddb.on('close')`

Emitted when this `ddb` has been closed. All `Ddbs` are automatically closed when the stream ends or is destroyed.

#### `ddb.close()`

Close this ddb. You only need to call this if you are sharing a lot of `Ddbs` and want to garbage collect some old unused ones.

#### `ddb.destroy(err)`

An alias to `stream.destroy`.

## Wire protocol

The dDatabase protocol uses a basic varint length prefixed format to send messages over the wire.

All messages contains a header indicating the type and `Ddb id`, and a protobuf encoded payload.

```
message = header + payload
```

A header is a varint that looks like this

```
header = numeric-ddb-id << 4 | numeric-type
```

The `Ddb id` is just an incrementing number for every Ddb shared and the type corresponds to which protobuf schema should be used to decode the payload.

The message is then wrapped in another varint containing the length of the message

```
wire = length(message) + message + length(message2) + message2 + ...
```

# dDatabase Documentation

## Introduction

## Features

* Thin replication. Only download the data you are interested in.
* Realtime. Get the latest updates to the log fast and securely.
* Performant. Uses a simple flat file structure to maximize I/O performance.
* Secure. Uses signed merkle trees to verify database integrity in real time.
* Browser support. Simply pick a storage provider (like [@dwcore/rem](https://github.com/dwcore/rem)) that works in the browser

## Usage

``` js
var ddatabase = require('@ddatabase/core')
var ddb = ddatabase('./ddbs', {valueEncoding: 'utf-8'})

ddb.append('greetings')
ddb.append('martian', function (err) {
  if (err) throw err
  ddb.get(0, console.log) // prints greetings
  ddb.get(1, console.log) // prints martian
})
```

## Terminology

 - **database**. This is what ddatabases are: distributed databases or `Ddbs`. Ddbs are permanent data structures that can be shared on the dWeb network.
 - **stream**. Streams are a tool in the code for reading or writing data. Streams are temporary and almost always returned by functions.
 - **tunnel**. Streams tend to either be readable (giving data) or writable (receiving data). If you connect a readable to a writable, that's called piping.
 - **fork stream**. A stream returned by the `fork()` function which can be piped to a peer. It is used to sync the peers' ddatabase ddbs.
 - **flocking**. Flocking describes adding yourself to the network, finding peers, and sharing data with them. Piping a replication ddb describes sharing the data with one peer.

## API

#### `var ddb = ddatabase(storage, [key], [options])`

Create a new ddatabase ddb.

`storage` should be set to a directory where you want to store the data and ddb metadata.

``` js
var ddb = ddatabase('./directory') // store data in ./directory
```

Alternatively you can pass a function instead that is called with every filename `ddatabase` needs to function and return your own [abstract-random-access](https://github.com/juliangruber/abstract-random-access) instance that is used to store the data.

``` js
var dwRem = require('@dwcore/rem')
var ddb = ddatabase(function (filename) {
  // filename will be one of: data, thinbit, tree, signatures, key, secret_key
  // the data file will contain all your data concatenated.

  // just store all files in dwRem by returning a dwrem instance
  return dwRem()
})
```

Per default `ddatabase` uses [dwRef](https://github.com/distributedweb/dwref). This is also useful if you want to store specific files in other directories. For example you might want to store the secret key elsewhere.

`key` can be set to a `ddatabase` ddb public key. If you do not set this the public key will be loaded from storage. If no key exists a new key pair will be generated.

`options` include:

``` js
{
  createIfMissing: true, // create a new `ddatabase` key pair if none was present in storage
  overwrite: false, // overwrite any old `ddatabase` that might already exist
  valueEncoding: 'json' | 'utf-8' | 'binary', // defaults to binary
  thin: false, // do not mark the entire ddb to be downloaded
  secretKey: buffer, // optionally pass the corresponding secret key yourself
  storeSecretKey: true, // if false, will not save the secret key
  storageCacheSize: 65536, // the # of entries to keep in the storage system's LRU cache (false or 0 to disable)
  onwrite: (index, data, peer, cb) // optional hook called before data is written after being verified
}
```

#### `ddb.writable`

Can we append to this ddb?

Populated after `ready` has been emitted. Will be `false` before the event.

#### `ddb.readable`

Can we read from this ddb? After closing a ddb this will be false.

Populated after `ready` has been emitted. Will be `false` before the event.

#### `ddb.key`

Buffer containing the public key identifying this ddb.

Populated after `ready` has been emitted. Will be `null` before the event.

#### `ddb.revelationKey`

Buffer containing a key derived from the ddb.key.
In contrast to `ddb.key` this key does not allow you to verify the data but can be used to announce or look for peers that are sharing the same ddb, without leaking the ddb key.

Populated after `ready` has been emitted. Will be `null` before the event.

#### `ddb.length`

How many blocks of data are available on this ddb?

Populated after `ready` has been emitted. Will be `0` before the event.

#### `ddb.byteLength`

How much data is available on this ddb in bytes?

Populated after `ready` has been emitted. Will be `0` before the event.

#### `ddb.get(index, [options], callback)`

Get a block of data.
If the data is not available locally this method will prioritize and wait for the data to be downloaded before calling the callback.

Options include

``` js
{
  wait: true, // wait for index to be downloaded
  timeout: 0, // wait at max some milliseconds (0 means no timeout)
  valueEncoding: 'json' | 'utf-8' | 'binary' // defaults to the ddb's valueEncoding
}
```

Callback is called with `(err, data)`

#### `ddb.getBatch(start, end, [options], callback)`

Get a range of blocks efficiently. Options include

``` js
{
  wait: sameAsAbove,
  timeout: sameAsAbove,
  valueEncoding: sameAsAbove
}
```

#### `ddb.head([options], callback)`

Get the block of data at the tip of the ddb. This will be the most recently
appended block.

Accepts the same `options` as `ddb.get()`.

#### `ddb.download([range], [callback])`

Download a range of data. Callback is called when all data has been downloaded.
A range can have the following properties:

``` js
{
  start: startIndex,
  end: nonInclusiveEndIndex,
  linear: false // download range linearly and not randomly
}
```

If you do not mark a range the entire ddb will be marked for download.

If you have not enabled thin mode (`thin: true` in the ddb constructor) then the entire
ddb will be marked for download for you when the ddb is created.

#### `ddb.undownload(range)`

Cancel a previous download request.

#### `ddb.signature([index], callback)`

Get a signature proving the correctness of the block at index, or the whole stream.

Callback is called with `(err, signature)`.
The signature has the following properties:
``` js
{
  index: lastSignedBlock,
  signature: Buffer
}
```

#### `ddb.verify(index, signature, callback)`

Verify a signature is correct for the data up to index, which must be the last signed
block associated with the signature.

Callback is called with `(err, success)` where success is true only if the signature is
correct.

#### `ddb.rootHashes(index, callback)`

Retrieve the root *hashes* for given `index`.

Callback is called with `(err, roots)`; `roots` is an *Array* of *Node* objects:
```
Node {
  index: location in the merkle tree of this root
  size: total bytes in children of this root
  hash: hash of the children of this root (32-byte buffer)
}
```


#### `var number = ddb.downloaded([start], [end])`

Returns total number of downloaded blocks within range.
If `end` is not specified it will default to the total number of blocks.
If `start` is not specified it will default to 0.

#### `var bool = ddb.has(index)`

Return true if a data block is available locally.
False otherwise.

#### `var bool = ddb.has(start, end)`
Return true if all data blocks within a range are available locally.
False otherwise.

#### `ddb.append(data, [callback])`

Append a block of data to the ddb.

Callback is called with `(err)` when all data has been written or an error occurred.

#### `ddb.clear(start, [end], [callback])`

Clear a range of data from the local cache.
Will clear the data from the thinbit and make a call to the underlying storage provider to delete the byte range the range occupies.

`end` defaults to `start + 1`.

#### `ddb.seek(byteOffset, callback)`

Seek to a byte offset.

Calls the callback with `(err, index, relativeOffset)`, where `index` is the data block the byteOffset is contained in and `relativeOffset` is
the relative byte offset in the data block.

#### `ddb.update([minLength], [callback])`

Wait for the ddb to contain at least `minLength` elements.
If you do not provide `minLength` it will be set to current length + 1.

Does not download any data from peers except for a proof of the new ddb length.

``` js
console.log('length is', ddb.length)
ddb.update(function () {
  console.log('length has increased', ddb.length)
})
```

#### `var stream = ddb.createReadStream([options])`

Create a readable stream of data.

Options include:

``` js
{
  start: 0, // read from this index
  end: ddb.length, // read until this index
  snapshot: true, // if set to false it will update `end` to `ddb.length` on every read
  tail: false, // sets `start` to `ddb.length`
  live: false, // set to true to keep reading forever
  timeout: 0, // timeout for each data event (0 means no timeout)
  wait: true // wait for data to be downloaded
}
```

#### `var stream = ddb.createWriteStream()`

Create a writable stream.

#### `var stream = ddb.replicate([options])`

Create a replication stream. You should pipe this to another ddatabase instance.

``` js
// assuming we have two ddbs, localDdb + remoteDdb, sharing the same key
// on a server
var net = require('net')
var server = net.createServer(function (socket) {
  socket.pipe(remoteDdb.replicate()).pipe(socket)
})

// on a client
var socket = net.connect(...)
socket.pipe(localDdb.replicate()).pipe(socket)
```

Options include:

``` js
{
  live: false, // keep replicating after all remote data has been downloaded?
  download: true, // download data from peers?
  encrypt: true // encrypt the data sent using the ddatabase key pair
}
```

#### `ddb.close([callback])`

Fully close this ddb.

Calls the callback with `(err)` when all storage has been closed.

#### `ddb.on('ready')`

Emitted when the ddb is ready and all properties have been populated.

#### `ddb.on('error', err)`

Emitted when the ddb experiences a critical error.

#### `ddb.on('download', index, data)`

Emitted when a data block has been downloaded.

#### `ddb.on('upload', index, data)`

Emitted when a data block is uploaded.

#### `ddb.on('append')`

Emitted when the ddb has been appended to (i.e. has a new length / byteLength)

#### `ddb.on('sync')`

Emitted every time ALL data from `0` to `ddb.length` has been downloaded.

#### `ddb.on('close')`

Emitted when the ddb has been fully closed

# dWeb's Flock Server Library


## Introduction

A tcp/utp `flock server` that auto announces itself using [@flockcore/channel](http://open.benchx.io/flockcore/channel). Basically a server-only version of [@flockcore/revelation](http://open.benchx.io/flockcore/revelation)

## Usage

First launch a flock server

```js
var createFlockServer = require('@flockcore/server')

var flockServer = createFlockServer(function (socket) {
  console.log('New connection!')
  socket.write('Greetings, Martian!\n')
  socket.end()
})

flockServer.listen('greetings-martian-server', 8080, function () {
  console.log('Now listening ...')
})
```

Then using `@flockcore/channel` you can connect to it

```js
var flockcoreChannel = require('@flockcore/channel')()
var net = require('net')

flockcoreChannel.on('peer', function (key, peer) {
  var socket = net.connect(peer.port, peer.host)

  socket.on('data', function (data) {
    process.stdout.write(data)
    socket.on('end', function () {
      process.exit()
    })
  })

  socket.on('error', function () {
    socket.destroy()
  })
})

flockcoreChannel.join('greetings-martian-server')
```

## API

#### `var flockServer = createFlockServer([revelationOptions], [onconnection])`

Create a new tcp + utp flock server. `revelationOptions` are forwarded to the [@flockcore/channel](http://open.benchx.io/channel) constructor. In addition you can set `{utp: false}` to disable utp and only use tcp

Optionally you can pass a `onconnection` listener

#### `flockServer.listen(key, [port], [onlistening])`

Start listening and announce your ip:port using the given key.

#### `flockServer.close([onclose])`

Completely close down the flock server.

#### `flockServer.join(key)`

Announce to an addition key after listening. It is also safe to call `.listen` multiple times instead.

#### `flockServer.address()`

Returns an address object after the flock server has started listening containing the port and local address.

#### `flockServer.leave(key)`

Stop announcing a key.

#### `flockServer.on('connection', connection, info)`

Emitted when a tcp or utp socket connects. The info object looks like this `{type: 'utp' | 'tcp'}` and can be used to figure out what kind of connection you received.

#### `flockServer.on('listening')`

Emitted when the flock server is fully listening.

#### `flockServer.on('close')`

Emitted when the flock server is fully closed.

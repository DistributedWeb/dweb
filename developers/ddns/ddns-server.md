# dDNS Server

## Introduction
It's easy to launch a `dDNS Revelation Server` for your next Distributed Web project. You can do so by following the instructions below:

```
npm i -g @distdns/server@latest
ddns
```

This should give the following output:

```
$ddns
ddns [command]
  announce [name]
    --port=(port)
    --host=(optional host)
    --server=(optional revelation server)
    --domain=(optional authoritative domain)
  lookup [name]
    --server=(optional revelation server)
    --domain=(optional authoritative domain)
  listen
    --port=(optional port)
    --ttl=(optional ttl in seconds)
    --domain=(optional authoritative domain)
    --diag=(enable diagnostic server, optional port, default 3030)
    --diagpw=(optional password for diagnostic server)
```

dWeb dDNS Server For Peer Revelation Over The Distributed Web Using Regular and Multicast DNS Libraries.

```
npm install @distdns/server
```

## Usage

``` js
var revelation = require('@distdns/server')

var revelation1 = revelation()
var revelation2 = revelation()

revelation1.on('peer', function (name, peer) {
  console.log(name, peer)
})

// announce an app
revelation2.announce('dstatus', 6620)
```

## API

#### `var rev = revelation([options])`

Create a new revelation instance. Options include:

``` js
{
  server: 'revelation1.dwebs.io:6620', // put a centralized DNS revelation server here
  ttl: someSeconds, // ttl for records in seconds. defaults to Infinity.
  limit: someLimit, // max number of records stored. defaults to 10000.
  multicast: true, // use @distdns/core. defaults to true.
  domain: 'dwebs.io', // top-level domain to use for records. defaults to dweb.local
  socket: someUdpSocket, // use this udp socket as the client socket
  loopback: false // Revelate yourself over Multicast
}
```

If you have more than one revelation server you can specify an array

``` js
{
  server: [
    'revelation1.dwebs.io:6620',
    'revelation2.dwebs.io:6620'
  ]
}
```

#### `dwRevelation.lookup(name, [callback])`

Do a lookup for a specific app name. When new peers are `revelated` for this name peer events will be emitted. The callback will be called when the query is complete.

``` js
dwRevelation.on('peer', function (name, peer) {
  console.log(name) // app name this peer was `revelated` for (ie 'example')
  console.log(peer) // {host: 'some-ip', port: 1234}
})
dwRevelation.lookup('example')
```

#### `dwRevelation.announce(name, port, [options], [callback])`

Announce a new port for a specific app name. Announce also does a lookup so you don't need to do that afterwards.

If you want to specify a public port (a port that is reachable from outside your firewall) you can set the `publicPort: port`
option. This will announce the public port to your list of DNS servers and use the other port over multicast.

You can also set `impliedPort: true` to announce the public port of the DNS socket to the list of DNS servers.

#### `dwRevelation.unannounce(name, port, [options], [callback])`

Stop announcing a port for an app. Has the same options as .announce

#### `dwRevelation.listen([port], [callback])`

Listen for DNS records on a specific port. You *only* need to call this if you want to turn your peer into a revelation server that other peers can use to store peer objects on.

``` js
var server = revelation()
server.listen(6620, function () {
  var dwRevelation = revelation({server: 'localhost:6620'})
  dwRevelation.announce('test-app', 8080) // will announce this record to the above revelation server
})
```

You can setup a revelation server to announce records on the internet as MultiCast DNS only works on a local network.
The port defaults to `53` which is the standard DNS port. Additionally it tries to bind to `5300` to support networks that filter DNS traffic.

#### `dwRevelation.destroy([onclose])`

Destroy the revelation instance. Will destroy the underlying udp socket as well.

#### `Event: "listening"`

Emitted after a successful `listen()`.

#### `Event: "close"`

Emitted after a successful `destroy()`.

#### `Event: "peer" (name, {host, port})

Emitted when a peer has been `revelated`.

 - **name** The app name the peer was `revelated` for.
 - **host** The address of the peer.
 - **port** The port the peer is listening on.

#### `Event: "announced" (name, {port})`

Emitted after a successful `announce()`.

 - **name** The app name that was announced.
 - **port** The port that was announced.

#### `Event: "unannounced" (name, {port})`

Emitted after a successful `unannounce()`.

 - **name** The app name that was unannounced.
 - **port** The port that was unannounced.

#### `Event: "traffic" (type, details)`

Emitted when any kind of message event occurs. The `type` will be prefixed with `'in:'` to indicate inbound, and `'out:'` to indicate outbound messages. This event is mostly useful for debugging.

#### `Event: "secrets-rotated"`

Emitted when the internal secrets used to generate session tokens have been rotated. This event is mostly useful for debugging.

#### `Event: "error" (err)`

Emitted when networking errors occur, such as failures to bind the socket (EACCES, EADDRINUSE).

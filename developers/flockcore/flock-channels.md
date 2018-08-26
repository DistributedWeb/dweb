# Flock Channels

## Introduction

Search for a key across multiple dWeb revelation networks and find peers who answer.

Currently searches across and advertises on [the Bittorrent DHT](https://en.wikipedia.org/wiki/Mainline_DHT), centralized dWeb DNS servers and [Multicast DNS](https://en.wikipedia.org/wiki/Multicast_DNS) simultaneously.


## Usage

### `var DC = require('@flockcore/channel')`

Returns a constructor

### `var fchannel = DC(<opts>)`

Returns a new instance. `opts` is optional and can have the following properties:

- `dns` - default `undefined`, if `false` will disable `dns` revelation, any other value type will be passed to the `@flockcore/server` constructor
- `dht` - default `undefined`, if `false` will disable `dht` revelation, any other value type will be passed to the `bittorrent-dht` constructor
- `hash` - default `sha1`. provide a custom hash function to hash ids before they are stored in the dht / on dns servers.

By default hashes are re-announced around every 10 min on the DHT and 1 min using DNS. Set `dht.interval` or `dns.interval` to change these.

### `fchannel.join(id, [port], [cb])`

Perform a lookup across all networks for `id`. `id` can be a buffer or a string.
Specify `port` if you want to announce that you share `id` as well.

If you specify `cb`, it will be called **when the first round** of revelation has completed. But only on the first round.

### `fchannel.leave(id, [port])`

Stop looking for `id`. `id` can be a buffer or a string.
Specify `port` to stop announcing that you share `id` as well.

### `fchannel.update()`

Force announce / lookup all joined hashes

### `var list = fchannel.list()`

List all the channels you have joined. The returned array items look like this

``` js
{
  id: <Buffer>,
  port: <port you are announcing>
}
```

### `fchannel.on('peer', id, peer, type)`

Emitted when a peer answers your query.

- `id` is the id (as a buffer) this peer was revelated for
- `peer` is the peer that was revelated `{port: port, host: host}`
- `type` is the network type (one of `['dht', 'dns']`)

### `fchannel.destroy(cb)`

Stops all lookups and advertisements and call `cb` when done.

### `fchannel.on('close')`

Emitted when the channel is destroyed

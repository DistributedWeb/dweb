# dWeb's Flock Network Presets

## Introduction

Use dWeb presets for dWeb `dns` and BitTorrent `dht` servers in [@flockcore/core](https://github.com/flockcore/core) or [@flockcore/revelation](https://github.com/flockcore/revelation). The *dWeb dDNS* and *dWeb DHT* servers are used to revelate other peers.

## Usage

Create a config object and pass it to a `Revelation Flock`.

Any options you specify will overwrite the defaults. See `Revelation Flock` for options.

```javascript
var Flock = require('@flockcore/revelation')
var presets = require('@flockcore/presets')

var config = presets({
  stream: function () {
    return drive.createPeerStream()
  }
})
var flock = Flock(config)
```

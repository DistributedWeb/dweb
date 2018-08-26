# dWeb Address Resolving

## Introduction

Library For Resolving dWeb URLs and dPack Keys


### Supports

* Common dWeb/dPack/dDrive key representations (`dweb://`, etc.)
* URLs with keys in them (`dwebs.io/6161616161616161616161616161616161616161616161616161616161616161`)
* `ddrive-key` or `dweb-key` headers
* Url to JSON http request that returns `{key: <dweb-key>}`
* dWeb DNS resolution (via [@distdns/core](/developer-tools/ddns/multicast-dns/))

## Install

```
npm install @dwebs/resolve
```

## Usage

```js
var dWebUrlResolve = require('@dwebs/resolve')

dWebUrlResolve(link, function (err, key) {
  console.log('found key', key)
})
```

## API

### `dWebUrlResolve(link, callback(err, key))`

Link can be string or buffer.

Resolution order:

1. Validate buffers or any strings with 64 character hashes in them via [@dwebs/codec](https://github.com/distributedweb/codec)
2. Check headers in http request
3. Check JSON request response for `key`
4. dWeb-DNS resolution via [@distdns/core](https://github.com/distdns/core)

### Examples
Note that `@dwebs/resolve` also supports other methods, such as detection of dWeb, dPack or dDrive keys in paths and http headers.

#### Simplest
* Plain: 40a7f6b6147ae695bcbcff432f684c7bb5291ea339c28c1755896cdeb80bd2f9
* DNS: test.dhost.io

#### With version
* Plain: 40a7f6b6147ae695bcbcff432f684c7bb5291ea339c28c1755896cdeb80bd2f9+5
* DNS: test.dhost.io+5

#### With scheme
* https: https://40a7f6b6147ae695bcbcff432f684c7bb5291ea339c28c1755896cdeb80bd2f9/
* dWeb: dweb://test.dhost.io

#### With path
* https: 40a7f6b6147ae695bcbcff432f684c7bb5291ea339c28c1755896cdeb80bd2f9/path1
* dweb: test.dhost.io/path2

#### Combinations
* 40a7f6b6147ae695bcbcff432f684c7bb5291ea339c28c1755896cdeb80bd2f9+5/path3
* dweb://40a7f6b6147ae695bcbcff432f684c7bb5291ea339c28c1755896cdeb80bd2f9+5/path4
* https://40a7f6b6147ae695bcbcff432f684c7bb5291ea339c28c1755896cdeb80bd2f9/path5 [(^1)][]
* https://test.dhost.io+5/path6 [(^2)][]

### Notes
1. browsers expect http and https schemes with traditional hostname, not a dWeb key
2. browsers expect http and https schemes with traditional hostname, no +5 (version) support

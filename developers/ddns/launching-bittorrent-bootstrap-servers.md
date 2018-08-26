# Launching BitTorrent Bootstrap Nodes

## Introduction
The dWeb operates alongside BitTorrent's network by using BitTorrent [DHT](http://docs.dwebs.io/glossary#dht) Bootstrap nodes. There is a dWeb utility for launching your own private bootstrap nodes to be used privately or for public use-cases, where you want a DHT bootstrap node that is dedicated to your dSite(s) or dApp(s).

To install our BitTorrent Bootstrap Node deployer, follow the instructions below:

```
npm i -g @dwdht/bootstrap@latest
dbootstrap
```

This should render the following output:

```
$dbootstrap
dWeb DHT Bootstrap Node Listening On 49737
```

## Usage

A `dWeb Bootstrap Node` is just a normal DHT node, except it doesn't answer any custom queries and will
just forward you to other nodes in the network.

``` sh
dbootstrap --port=10000
```


## Developer Notes
After spinning up a node add it to your [@dwdht/rpc](https://github.com/dwdht/rpc) bootstrap list

``` js
var node = dWebDHT({
  bootstrap: ['mydomain.com:10000']
})

var stream = node.query({
  command: 'your-command',
  target: ...
})
```

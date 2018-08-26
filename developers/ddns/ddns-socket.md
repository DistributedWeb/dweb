# dDNS Socket
## Introduction

dWeb Library For Making DNS Requests.

```
npm install @distdns/socket
```

## Usage

``` js
const dWebDNS = require('@distdns/socket')
const dnsSocket = dWebDNS()

socket.query({
  questions: [{
    type: 'A',
    name: 'google.com'
  }]
}, 53, '8.8.8.8', (err, res) => {
  console.log(err, res) // prints the A record for google.com
})
```

## API

#### `var dnsSocket = dWebDNS([options])`

Create a new DNS socket instance. The `options` object includes:

- `retries` *Number*: Number of total query attempts made during `timeout`. Default: 5.
- `socket` *Object*: A custom dGram socket. Default: A `'udp4'` socket.
- `timeout` *Number*: Total timeout in milliseconds after which a `'timeout'` event is emitted. Default: 7500.

#### `dnsSocket.on('query', query, port, host)`

Emitted when a DNS query is received.

#### `dnsSocket.on('response', response, port, host)`

Emitted when a DNS response is received.

#### `var id = dnsSocket.query(query, port, [host], [callback])`

Send a DNS query. If host is omitted it defaults to localhost. When the remote replies the callback is called with `(err, response, query)` and an response is emitted as well. If the query times out the callback is called with an error.

Returns the query id

#### `dnsSocket.response(query, response, port, [host])`

Send a response to a query.

#### `dnsSocket.cancel(id)`

Cancel a query

#### `dnsSocket.bind(port, [onlistening])`

Bind the underlying UDP socket to a specific port.

#### `dnsSocket.destroy([onclose])`

Destroy the socket.

#### `dnsSocket.inflight`

Number of inflight queries.

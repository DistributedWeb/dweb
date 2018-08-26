# MultiCast DNS Types
## Introduction

dWeb Library For MultiCast DNS Types, allowing for the parsing and stringifying of mDNS Service Types.

```
npm install @distdns/types
```

## Usage

``` js
var mDNSTypes = require('@distdns/types')

console.log(mDNSTypes.stringify({name: 'http', protocol: 'tcp', subtypes: ['sub1', 'sub2']})) // _http._tcp._sub1._sub2
console.log(mDNSTypes.parse('_http._tcp._sub1._sub2')) // {name: 'http', protocol: 'tcp', subtypes: ['sub1', 'sub2']}
```

The following shorthands also exist

``` js
mDNSTypes.stringify(name, protocol, subtypes)
mDNSTypes.tcp(name, subtypes) // set protocol to tcp
mDNSTypes.udp(name, subtypes) // set protocol to udp
```

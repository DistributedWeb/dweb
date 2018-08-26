# dDNS Lookups 

## Introduction

dWeb dDNS Library For MultiCast DNS Hostname>IP Lookups.

```
npm install @distdns/lookup
```

OSX supports resolving .local domains using mDNS per default but unfortunately not
all other platforms have as good support for it. On Ubuntu you currently have to install and run the `avahi-daemon`
which is why I wrote this module that allows you to always resolve them from javascript without relying on os support.

## Usage

``` js
var lookup = require('@distdns/lookup')

lookup('google.com', function (err, ip) {
  console.log(ip) // is resolved using normal DNS (173.194.116.32 on my machine)
})

lookup('dweb.local', function (err, ip) {
  console.log(ip) // is resolved using MultiCast DNS (192.168.10.254 on my machine)
})
```

If you want the node core DNS module to always support MultiCast DNS you can do the following

``` js
require('@distdns/lookup/global')
var dns = require('dns')

dns.lookup('dweb.local', function (err, ip) {
  console.log(ip) // is resolved using MultiCast DNS (192.168.10.254 on my machine)
})
```

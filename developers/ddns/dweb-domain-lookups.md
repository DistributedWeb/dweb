# dWeb Domain Lookups In dApps

## Introduction
Library For [dWeb](https://dwebs.io) Vault Lookups Over HTTP. Enables dWeb Shortnames.

## Installation

### Install With NPM
```
npm install @dwebs/dns
```

### Install With YARN
```
yarn add @dwebs/dns
```

## API

```js
var dWebDNS = require('@dwebs/dns')()

// resolve a name: pass the hostname by itself
dWebDns.dWebResolveAddress('dwebs.io', function (err, key) { ... })
dWebDns.dWebResolveAddress('dwebs.io').then(key => ...)

// dont use cached 'misses'
dWebDns.dWebResolveAddress('dwebs.io', {ignoreCachedMiss: true})

// dont use the cache at all
dWebDns.dWebResolveAddress('dwebs.io', {ignoreCache: true})

// dont use dns-over-https
dWebDns.dWebResolveAddress('dwebs.io', {noDnsOverHttps: true})

// dont use .well-known/dweb
dWebDns.dWebResolveAddress('dwebs.io', {dWebNotWellKnown: true})

// list all entries in the cache
dWebDns.listCache()

// clear the cache
dWebDns.flushCache()

// configure the DNS-over-HTTPS host used
var dWebDns = require('@dwebs/dns')({
  dWebDnsHost: 'dns.google.com',
  dWebDnsHostPath: '/resolve'
})

// use a persistent fallback cache
// (this is handy for persistent dns data when offline)
var dWebDns = require('@dwebs/dns')({
  persistentCache: {
    read: async (name, err) => {
      // try lookup
      // if failed, you can throw the original error:
      throw err
    },
    write: async (name, key, ttl) => {
      // write to your cache
    }
  }
})
```

**Option 1 (DNS-over-HTTPS).** Create a DNS TXT record with he following schema:

```
dwebkey={key}
```

**Option 2 (.well-known/dweb).** Place a file at `/.well-known/dweb` with the following schema:

```
{dweb-url}
TTL={time in seconds}
```

TTL is optional and will default to `3600` (one hour). If set to `0`, the entry is not cached.

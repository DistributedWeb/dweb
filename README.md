# The Distributed Web

- [dWeb Glossary](#dweb-glossary)
- [dWeb Protocol](#dweb-protocol)
  - [dWeb Network](#dweb-network)
    - [Important dWeb Network Information](#important-dweb-network-information)
  - [dWeb Codec](#dweb-codec)
  - [dWeb Cryptography](#dweb-cryptography)  
  - [dWeb Core Libraries](#dweb-core-libraries)
  - [dWeb URLs](#dweb-urls)
- [Become a dWeb Developer](#become-a-dweb-developer)
  - [Developer Prerequisites](#developer-prerequisites)
    - [Install Node.js](install-node-js)
      - [Installing Node.js On Linux and Mac](#install-node-js)
      - [Installing Node.js On Windows](#install-node-js)
  - [Developer Tools](#developer-tools)
    - [dDNS Testing Utility](#ddns-testing-utility)
    - [Checking Network Health](#checking-network-health)
    - [dPack CLI](#dpack-cli)
    - [Launching dWeb Towers](#launching-dweb-towers)
    - [dDNS Utilities](#ddns-utilities)
      - [dDNS Revelation Server](#ddns-revelation-server)
      - [dDNS Register Utility](#ddns-register-utility)
      - [dDNS Lookup Utility](#ddns-lookup-utility)
    - [Launching BitTorrent Bootstrap Nodes](#launching-bittorrent-bootstrap-nodes)
    - [Distributed Code Editor](#distributed-code-editor)
    - [dPack Metadata](#dpack-metadata)
    - [dSiteDB](#dsitedb)
    - [dDrives](#ddrives)
    - [dDatabases](#ddatabases)
    - [dWeb Computer](#dweb-computer)
- [Public dWeb Infrastructure](#public-dweb-infrastructure)
  - [Public dWeb Distributed DNS Servers](#public-dweb-distributed-dns-servers)
  - [Public dWeb BitTorrent Bootstrap Servers](#public-dweb-bittorrent-bootstrap-servers)
  - [Public dWeb Towers](#public-dweb-towers)
  - [Public dPack Repository](#public-dpack-repository)
    - [Emergency Public dPack Repository Nodes](#emergency-public-dpack-repository-nodes)
    - [Running Your Own dPack Repository](#running-your-own-dpack-repository)
  - [Public Dr. Satoshi Nodes](#public-dr-satoshi-nodes)
- [Browsing The dWeb](#browsing-the-dweb)
  - [Compatible Web Browsers](#compatible-web-browsers)
  - [Popular dWeb dSites and dApps ](#popular-dweb-dsites-and-dapps)
    - [dHosting](#dhosting)
    - [dWebs](#dwebs)
    - [dBrowser](#dbrowser)

## dWeb Glossary

- `dpack` - Distributed Web's version of `git` and `npm` as a combined solution. Also known as a distributed package. [Wiki definition](https://wiki.dwebs.io/dpack)
- `ddns` or `distdns` - Distributed Domain Name System. Used across most of dWeb's applications and libraries. [Wiki definition](https://wiki.dwebs.io/ddns)
- `dweb` - Protocol for the Distributed Web. Like `http:// `, `dweb://` is used to access dWeb-based resources via a `Distributed Web Browser`. [Wiki definition](https://wiki.dwebs.io/dweb)
- `distributed web browser` or `dbrowser` - A web browser that is able to access `dweb://` URLs and `dPacks` [Wiki definition](https://wiki.dwebs.io/dbrowser)
- `dweb tower` or `tower` - A beaming node used by dApps and dSites to beam data across the distributed web. [Wiki definition](https://wiki.dwebs.io/dweb_tower)
- `revelation` or `discovery` - The method of peers finding other peers/flocks [Wiki definition](https://wiki.dwebs.io/revelation)
- `flock` - A gathering of peers on the distributed web. [Wiki definition](https://wiki.dwebs.io/flock)
- `channel` - Used by a flock to find other flocks. [Wiki definition](https://wiki.dwebs.io/channel)
- `distributed streams` - Distributed data streams, used in `dApps` and `dSites` [Wiki definition](https://wiki.dwebs.io/distributed_streams)
- `dsite` - Decentralized and distributed websites. [Wiki definition](https://wiki.dwebs.io/dsite)
- `dapp` - Decentralized and distributed apps. [Wiki definition](https://wiki.dwebs.io/dapp)
- `blockchain` - Distributed ledger that requires the central web to function. [Wiki definition](https://wiki.dwebs.io/blockchain)
- `p2p` - Stands for peer-to-peer. [Wiki definition](https://wiki.dwebs.io/p2p)
- `peers` - [Wiki definition](https://wiki.dwebs.io/peers)
- `dstatus` - A distributed and decentralized version of Twitter for the dWeb [Wiki definition](https://wiki.dwebs.io/dstatus)
- `dtunes` - A distributed and decentralized version of Spotify for the dWeb [Wiki definition](https://wiki.dwebs.io/dtunes)
- `meteorIDE` - A distributed and decentralized code editor for benOS and dBrowser [Wiki definition](https://wiki.dwebs.io/meteorIDE)

## dWeb Protocol
The dWeb protocol is a combination of many protocols and consistently growing with more features. dWeb stands for "Distributed Web" and operates through the dweb:// web address prefix, where dSites (Distributed websites) and dApps (Distributed apps) can be accessed. There is a lot of software that allows the dWeb to grow and many toolsets for developers to build dSites/dApps and distribute them to peers around the world. dWeb represents the modern-day HTTP/HTTPS protocols, tied into one. Below are tons of resources and guides to get you started on the Distributed Web. 

- ***Note: This guide is constantly growing and information is constantly being added here. If you find any errors, please let us know, by emailing us at docs@dwebs.io***

### dWeb Network 
The dWeb network is completely peer-to-peer. Although we do have some central infrastructure out there, that central infrastructure is peered and all of our websites exist on the dweb:// 

#### Important dWeb Network Information 

- **Address Resolving** 
   - Addresses always start with **dweb://**
   - Addresses are always 64 bit hashes
   - dweb:// based dSites and dApps can be accessed via their normal TLDs. ***ex: dweb://dwebs.io***

- **dWeb Global Network Ports**
  - Global DNS Revelation - Port 6620 
  - Dr. Satoshi (Network Health Peers) - Port 6621 
  - BitTorrent DHT 
  - BenchCore (Root Payments Protocol / Network) - Port 6620 
 
- **Serving Over HTTP**
  - Requires a special **TXT** record **@** the base level of the TLD being used. See more [here](#serve-over-http)
  - Always uses the **DWEBKEY** variable in the **TXT** record. 
 
### dWeb Codec 
dWeb addresses encoded and decoded via a special codec library [@dwebs/codec](https://npmjs.org/package/@dwebs/codec) that can be included in dSites and dApps for extended functionality. 

**Usage Example:**
```
var encoding = require('@dwebs/codec')

var link = 'd50b3af6e2bf329cfe8cbfa9b62e811c15ba5ac6414d2961f833387de0b2b704'
var buf = encoding.decode(link)
console.log('%s -> %s', link, buf)
console.log('%s -> %s', buf, encoding.encode(buf))
```

### dWeb Cryptography 
dWeb, although not powered by any sort of blockchain-based software - does utilize [Sodium's](https://github.com/jedisct1/libsodium) cryptographic libraries for addresses, [dPack](https://dpack.io) keys and more. 

### dWeb Core Libraries 
The dWeb protocol (dweb://) is powered by many core libraries that can also be used within your dSites and dApps! Below are repository links, NPM links and current versioning information for each of the core libraries that power the dWeb. 

| Package | Version | Description
|---|---|---|
| [buffer-key](https://github.com/distributedweb/buffer-key) | [![npm](https://img.shields.io/npm/v/@dwebs/buffer-key.svg)](https://www.npmjs.com/package/@dwebs/buffer-key) | dWeb Key Buffering |
| [codec](https://github.com/distributedweb/codec) | [![npm](https://img.shields.io/npm/v/@dwebs/codec.svg)](https://www.npmjs.com/package/@dwebs/codec) | dWeb Key Codec |
| [dns](https://github.com/distributedweb/) | [![npm](https://img.shields.io/npm/v/@dwebs/dns.svg)](https://www.npmjs.com/package/@dwebs/dns) | dWeb DNS Library |
| [dsitedb](https://github.com/distributedweb/dsitedb) | [![npm](https://img.shields.io/npm/v/@dwebs/dsitedb.svg)](https://www.npmjs.com/package/@dwebs/dsitedb) | dSite/dApp Distributed Database Library |
| [gateway](https://github.com/distributedweb/gateway) | [![npm](https://img.shields.io/npm/v/@dwebs/gateway.svg)](https://www.npmjs.com/package/@dwebs/gateway) | dWeb Network Gateway |
| [helper](https://github.com/distributedweb/helper) | [![npm](https://img.shields.io/npm/v/@dwebs/helper.svg)](https://www.npmjs.com/package/@dwebs/helper) | dWeb Network Helper |
| [parse](https://github.com/distributedweb/parse) | [![npm](https://img.shields.io/npm/v/@dwebs/parse.svg)](https://www.npmjs.com/package/@dwebs/parse) | dWeb Address Parsing |
| [peer](https://github.com/distributedweb/peer) | [![npm](https://img.shields.io/npm/v/@dwebs/peer.svg)](https://www.npmjs.com/package/@dwebs/peer) | dWeb P2P Library |
| [resolve](https://github.com/distributedweb/resolve) | [![npm](https://img.shields.io/npm/v/@dwebs/resolve.svg)](https://www.npmjs.com/package/@dwebs/resolve) | dWeb Address Resolver Library |
| [rpc-ws](https://github.com/distributedweb/rpc-ws) | [![npm](https://img.shields.io/npm/v/@dwebs/rpc-ws.svg)](https://www.npmjs.com/package/@dwebs/rpc-ws) | dWeb RPC Over Web Sockets Library |
| [tower](https://github.com/distributedweb/tower) | [![npm](https://img.shields.io/npm/v/@dwebs/tower.svg)](https://www.npmjs.com/package/@dwebs/tower) | dWeb Tower Launcher CLI |

### dWeb URLs
dWeb URLs operate just like HTTP/HTTPS-based URLs, with a few differences:

- Where on the central web a HTTP/HTTPS-based URL can serve network resources via a TLD or an IPV4/IPV6 address, dWeb-based URLs can serve network resources via a TLD or a dWeb Key. 

- dWeb addresses also start with the protocol prefix, just like `http://` or `https://`, dWeb addresses begin with the `dweb://` prefix. dWeb addresses are always secure with end-to-end encryption, so there is no such thing as an insecure prefix, as you would find on the central web with `http://`

### dWeb Financial Protocols 


#### BenchCore (BEX)

## Become a dWeb Developer

### Developer Prerequisites

#### Install Node.js

##### Installing Node.js On Linux and Mac
Using benOS' [getNJS](http://github.com/benchOS/getnjs) installer, you can install the latest version of Node.js on Linux/Mac machines. Simply paste the line below and everything will install automatically.

```
curl -fs https://raw.githubusercontent.com/benchOS/getnjs/master/go | sh && getnjs latest
```

##### Installing Node.js On Windows

```
Coming soon
```

###  Developer Tools

#### dDNS Testing Utility
We have a utility for testing dWeb's [Public dWeb Distributed DNS Servers](#public-dweb-distributed-dns-servers). Follow the directions below to test:

```
npm i -g @distdns/test@latest
ddns-test
```

Should output something like this:

```
$ddns-test
dWeb dDNS Is Working { port: 53321, host: '106.221.123.143' }
```

#### Checking Network Health  
The [dPack CLI](http://docs.dpack.io/cli) comes with a built-in utility known as [Dr. Satoshi](http://docs.dpack.io/dr-satoshi) for checking the health of the Distributed Web (dWeb) network. You can use Dr. Satoshi through the [dPack CLI](#dpack-cli) or you can install it as a standalone utility by following the directions below:

```
npm i -g @dpack/drsatoshi
satoshi
```

This should result in the following output:

```
$satoshi
Welcome to Dr Satoshi for dPack

Software Info:
  darwin x64
  Node v10.5.0
  Dr Satoshi vundefined

Which tests would you like to run?
> Basic dPack Tests (Checks your dPack installation and network setup)
  dWeb P2P Network Test (Debug connections between two computers)

```

For more information on how to use Dr. Satoshi, go to the [official Dr. Satoshi documentation](http://docs.dpack.io/dr-satoshi)


#### dPack CLI
dPack has an official command CLI tool for developers to create, distribute, test, fork, pull and push dPacks to any dPack-ready distributed repository. You can install the CLI by following the instructions below:

```
npm i -g @dpack/cli@latest
dpack
```

**dPack CLI Options**

```
$dpack

Usage: dpack <cmd> [<dir>] [options]

Create, Distribute and Sync Files:
   dpack create                  create empty dPack and dpack.json
   dpack dist                    create dPack, import files to the respository and distribute to the dWeb
   dpack sync                    import files to existing dpack & sync with network

Fork, Pull and Sync Files:
   dpack fork <link> [<dir>]     Fork a dPack via link to <dir>
   dpack pull                    Pull updates of a dPack & exit
   dpack dweb                    Sync dPack with the dWeb

dPack Info:
   dpack log                     Print a dPack's history
   dpack status                  Prints key and information about a local dPack

dPack Developer Management:
   dpack <cmd> [<repository>]    All commands take <repository> option
   dpack register                Register new repository account
   dpack login                   Login to repository account
   dpack publish                 Publish dPack to repository
   dpack whoami                  Print repository account information
   dpack logout                  Logout from current repository

dPack Shortcut Commands:
   dpack <link> [<dir>]          Fork dPack or Sync dPack in <dir> with dWeb
   dpack <dir>                   Create dPack and sync dPack in <dir> with dWeb

dPack Troubleshooting & Help:
   dpack satoshi                 Check with Dr. Satoshi on the current health of the dWeb
   dpack help                    Print dPack manpage
   dpack <command> --help, -h    Print specific usage parameters for a dPack command
   dpack --version, -v           Print current dPack version


Have fun using dPack! Learn more at docs.dpack.io
```

***dWeb Live Testing Links***
You may want to test `dpack sync` or `dpack fork` utilities to make sure everything is functioning correctly. To do so, we have several dWeb links that are live for testing. The following links should be valid and working at this time:

- [dweb://07eeb067dd5b444232964a97f0f2a19e60af8fd4b21fb09f38c7948967034bdc](dweb://07eeb067dd5b444232964a97f0f2a19e60af8fd4b21fb09f38c7948967034bdc) - Minimum dPack ( Just dpack.json / / 1 File )

- [dweb://42691e1c54a6e0a1ca4164ac5223adee488e60e4c75f549ed9d8368cc42e8fb6](dweb://42691e1c54a6e0a1ca4164ac5223adee488e60e4c75f549ed9d8368cc42e8fb6) - Small dPack ( / 1 File)

- [dweb://2b1b554f7b184fc10b8d61bae4c2c4248f56de2ef631a6778b2768914a559aa2](dweb://2b1b554f7b184fc10b8d61bae4c2c4248f56de2ef631a6778b2768914a559aa2) - Medium dPack ( / 2 Files)

- [dweb://d50b3af6e2bf329cfe8cbfa9b62e811c15ba5ac6414d2961f833387de0b2b704](dweb://d50b3af6e2bf329cfe8cbfa9b62e811c15ba5ac6414d2961f833387de0b2b704) - Large dPack (217MB / 26 Files)

- [dweb://1633766920c7b3f1e1918563a2fdab2c854f8f0fc6a5074156cc3d29f975f7dd](dweb://1633766920c7b3f1e1918563a2fdab2c854f8f0fc6a5074156cc3d29f975f7dd) - Website dPack (Complete Website / 171KB / 18 files)

** You can actually host your own test links for the community. If you would like to donate test dPack links, please email them to [tests@dpack.io](mailto:tests@dpack.io). We will check out the content and post if they fit our [Acceptable Use Policy](https://dhosting.io/acceptable-use-policy)

For more information on how to use dPack, go to the [Official dPack documentation](http://docs.dpack.io)

#### Launching dWeb Towers
When launching a `dSite` or a `dApp`, developers can use public dTowers to listen for events, subscriptions and beam broadcasts across the Distributed Web, or they can run their own dedicated towers for total decentralization. Below is a guide on how to launch your own towers. Towers run on `port 80` by default.

```
npm i -g @dwebs/tower@latest
dtower
```

This should result in the following output:

```
$dtower list
Usage: dtower listen|subscribe|broadcast
```

For more information on how to use dPack, go to the [official dWeb Tower documentation](http://docs.dwebs.io/tower)


#### dDNS Utilities

```
npm i -g @dwebs/tower@latest
dtower
```

This should result in the following output:

```
$dtower list
Usage: dtower listen|subscribe|broadcast
```

For more information on how to use dPack, go to the [official dWeb Tower documentation](http://docs.dwebs.io/tower)

##### dDNS Revelation Server
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

For more information on how to use dDNS revelation servers, go to the [official dWeb dDNS documentation](http://docs.dwebs.io/distributed-dns/server)

##### dDNS Register Utility

```
npm i -g @distdns/register@latest
dregister
```

This should render the following output:

```
$dregister
Usage: dregister [name]
```

For more information on how to use dDNS revelation servers, go to the [official dWeb dDNS registration documentation](http://docs.dwebs.io/distributed-dns/register)


##### dDNS Lookup Utility

```
npm i -g @distdns/lookup@latest
dlookup
```

This should render the following output:

```
$dlookup
Usage: dlookup [name]
```

For more information on how to use dDNS revelation servers, go to the [official dWeb dDNS lookup documentation](http://docs.dwebs.io/distributed-dns/lookup)

#### Launching BitTorrent Bootstrap Nodes
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

For more information on how to use dWeb-ready BitTorrent Bootstrap Nodes, go to the [official dWeb dwDHT bootstrap documentation](http://docs.dwebs.io/dwdht/bootstrap)

#### Distributed Code Editor

#### dPack Metadata

#### dSiteDB

#### dDrives

#### dDatabases

#### dWeb Computer

####

## Public dWeb Infrastructure

### Public dWeb Distributed DNS Servers
We maintain 6 public `DNS Revelation` servers on the `dwebs.io` domain. You are welcomed to use these servers for any of your distributed web projects.

- [http://revelation1.dwebs.io](http://revelation1.dwebs.io)
- [http://revelation2.dwebs.io](http://revelation2.dwebs.io)
- [http://revelation3.dwebs.io](http://revelation3.dwebs.io)
- [http://revelation4.dwebs.io](http://revelation4.dwebs.io)
- [http://revelation5.dwebs.io](http://revelation5.dwebs.io)
- [http://revelation6.dwebs.io](http://revelation6.dwebs.io)

### Public dWeb BitTorrent Bootstrap Servers
We maintain 6 public `BitTorrent Bootstrap` servers on the `dwebs.io` domain. You are welcomed to use these servers for any of your distributed web projects.

- [http://bootstrap1.dwebs.io](http://bootstrap1.dwebs.io)
- [http://bootstrap2.dwebs.io](http://bootstrap2.dwebs.io)
- [http://bootstrap3.dwebs.io](http://bootstrap3.dwebs.io)
- [http://bootstrap4.dwebs.io](http://bootstrap4.dwebs.io)
- [http://bootstrap5.dwebs.io](http://bootstrap5.dwebs.io)
- [http://bootstrap6.dwebs.io](http://bootstrap6.dwebs.io)


### Public dWeb Towers
We maintain 6 public `dWeb Tower` servers on the `dwebs.io` domain. You are welcomed to use these servers for any of your distributed web projects.

- [http://tower1.dwebs.io](http://tower1.dwebs.io)
- [http://tower2.dwebs.io](http://tower2.dwebs.io)
- [http://tower3.dwebs.io](http://tower3.dwebs.io)
- [http://tower4.dwebs.io](http://tower4.dwebs.io)
- [http://tower5.dwebs.io](http://tower5.dwebs.io)
- [http://tower6.dwebs.io](http://tower6.dwebs.io)

### Public dPack Repository
We maintain a public version of dPack's global and distributed repository. You can access it at the link below:

- [https://dpacks.io](https://dpacks.io)


#### Emergency Public dPack Repository Nodes
In the event the dPacks.io repository is taken down, you can access the dPack repository at several other domains

- [http://dpack1.dwtowers.network](http://dpack1.dwtowers.network)
- [http://dpack2.dwtowers.network](http://dpack2.dwtowers.network)
- [http://dpack3.dwtowers.network](http://dpack3.dwtowers.network)
- [http://dpack4.dwtowers.network](http://dpack4.dwtowers.network)
- [http://dpack5.dwtowers.network](http://dpack5.dwtowers.network)
- [http://dpack6.dwtowers.network](http://dpack6.dwtowers.network)
- [http://dpack7.dwtowers.network](http://dpack7.dwtowers.network)

#### Running Your Own dPack Repository

You can run your own private dPack repository or throw up a emergency dPack repository node to keep us #dwebStrong. In order to launch your own dPack repository, please follow the instructions on our [dPack custom repository guide](http://docs.dpacks.io/custom-repository).

We also run a public dPack repository [here](https://dpacks.io) and several emergency nodes you can find [here](#emergency-public-dpack-repository-nodes).

##### Creating Public/Private Keys For Configuration

**Private Key**
```js
openssl ecparam -genkey -name secp521r1 -noout -out ecdsa-p521-private.pem
```
**Public Key**
```js
openssl ec -in ecdsa-p521-private.pem -pubout -out ecdsa-p521-public.pem 
```
### Public Dr. Satoshi Nodes
The [dPack CLI](http://docs.dpack.io/cli) comes with a built-in utility called [Dr. Satoshi](http://docs.dpack.io/dr-satoshi) for checking the health of the Distributed Web (dWeb) network. You could easily test the network yourself, by using several Satoshi nodes that are publicly available and paid for by the [Distributed Webs](https://distributedwebs.org) non-profit project.

We maintain several publicly available Dr. Satoshi nodes. You can use any of these nodes in your own Dr. Satoshi build below:

- [http://satoshi.dwebs.io:6620](http://satoshi.dwebs.io:6620) - Fully functioning
- [http://satoshi2.dwebs.io:6620](http://satoshi2.dwebs.io:6620) - ***Down for maintenance***
- [http://satoshi3.dwebs.io:6620](http://satoshi3.dwebs.io:6620) - ***Down for maintenance***
- [http://satoshi4.dwebs.io:6620](http://satoshi4.dwebs.io:6620) - ***Down for maintenance***

## Browsing The dWeb


### Popular dWeb dSites and dApps 

#### dHosting 
dHosting provides distributed hosting services for dSites and dApps on the dweb://. Allowing developers to keep their dSites and dApps online forever, while also providing a shortname on the dhosting.io globally recognized TLD (top-level domain name), which can be used as a CNAME, to fully host your website on the dWeb, rather than paying for expensive hosting services. The [dHosting.io Website](https://dhosting.io) can be accessed from over dweb:// and https://. 

**dweb:// Links**
Coming soon.

**Served Over HTTP/HTTPS**
Coming soon.


#### dWebs
dWebs is the official website for the dweb:// protocol and has many dWeb (Distributed web) resoures for developers and users alike. The [dWebs.io Website](https://dwebs.io) can be accessed from over dweb:// and https://. 

**dweb:// Links**
- [dweb://49096d06943b4136db9dda54c8a31e427f5b66b33ebda17d4a15bea1caed73cc](dweb://49096d06943b4136db9dda54c8a31e427f5b66b33ebda17d4a15bea1caed73cc)
- [dweb://dwebs.io](dweb://dwebs.io)

**Served Over HTTP/HTTPS**
- [https://dwebs.io](https://dwebs.io)


#### dBrowser
**dweb:// Links**
- [dweb://13ca040c53fc8bcaa74de142449e30ece2b3576a8c9beff2ba3b13fb98f5e3c7](dweb://13ca040c53fc8bcaa74de142449e30ece2b3576a8c9beff2ba3b13fb98f5e3c7)
- [dweb://dbrowser.io](dweb://dbrowser.io)

**Served Over HTTP/HTTPS**
- [https://dbrowser.io](https://dbrowser.io)



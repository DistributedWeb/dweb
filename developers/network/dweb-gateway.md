# dWeb Gateway CLI

## Introduction

A configurable in-memory [dWeb](https://dwebs.io/)-to-HTTP gateway, so you can visit dWeb/dPack vaults from your browser.

If you want a browser that can visit dWeb/dPack vaults, check out [dBrowser](https://dbrowser.io/) or you can use the official dWeb [Firefox Webextension](https://dbrowser.io/downloads/firefox)

## Install

To get the `dweb-gateway` command for running your own gateway, use [npm](https://www.npmjs.com/):

```
npm i -g @dwebs/gateway
```

## Usage

You can run `dweb-gateway` to start a dWeb gateway server that listens on port 3000. You can also configure it! You can print usage information with `dweb-gateway -h`:

```
$ dweb-gateway -h
dweb-gateway

Options:
  --version     Show version number [boolean]
  --config      Path to JSON config file
  --port, -p    Port for the dWeb gateway to listen on. [default: 3000]
  --dir, -d     Directory to use as a cache.[string] [default: "~/.dweb-gateway"]
  --max, -m     Maximum number of vaults to serve at a time. [default: 20]
  --maxAge, -M  Number of milliseconds before vaults are removed from the cache. [default: 600000]
  -h, --help    Show help [boolean]
```

You can visit dPack/dWeb vaults through the dWeb gateway using a route like this:

```
http://localhost:3000/{dwebKey}/{path...}
```

For example:

```
http://localhost:3000/40a7f6b6147ae695bcbcff432f684c7bb5291ea339c28c1755896cdeb80bd2f9/img/test.png
```

The gateway will even resolve URLs using [@dwebs/dns](https://docs.dwebs.io/gateway/dweb-url-to-https-to-flock-revelation):

```
http://localhost:3000/test.dnames.io/img/test.png
```

# dWeb Address Key Codec

## Table of Contents

- [Background](#background)
- [Installation](#installation)
- [Usage](#usage)
- [API](#api)

## Background
[dWeb's](https://dwebs.io/) way of encoding and decoding dWeb keys. Which are in turn, dWeb addresses.

## Installation

### NPM Install

```
npm install @dwebs/codec
```

### Yarn Installation

```
yarn add @dwebs/codec
```

## Usage

```js
var dWebCreateAddress = require('@dwebs/codec')

var dWebLink = '25097091325097097135098983593802'
var dWebLinkBuffer = dWebAddressEncode.dWebAddressDecode(dWebLink)
console.log('%s -> %s', dWebLink, dWebLinkBuffer)
console.log('%s -> %s', dWebLinkBuffer, dWebCreateAddress.dWebAddressEncode(dWebLinkBuffer))
```

## API

### .dWebAddressEncode(buf)
### .toStr(buf)

Encode `buf` into a hex string. Throws if `buf` isn't 32 bytes of length.

If `buf` is already a string, checks if it's valid and returns it.

### .dWebAddressDecode(str)
### .toBuf(str)

Decode `str` into its binary representation.

If `str` is already a buffer, checks if it's valid and returns it.

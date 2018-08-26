# dWeb Address Key Buffering

## Introduction
This library `might` return a dWeb key in a buffered format.

## Installation

```
npm install @dwebs/buffer-key
```

```
yarn add @dwebs/buffer-key
```

This library `might` return a dWeb key in a buffered format.

* **valid string/buffer**: Returns a dWeb key as a buffer
* **invalid string**: Throws error
* **null**: Returns null

Check out [@dwebs/convert-key](https://github.com/distributedweb/convert-key) to get keys as strings or buffers!

## Usage

```js
var dWebBufferKey = require('@dwebs/buffer-key')

var dWebKey = dWebBufferKey(process.argv[2])

try {
  console.log('maybe a key: key valid or null', dWebBufferKey(dWebKey))
} catch (e) {
  console.log('key not valid')
}
```

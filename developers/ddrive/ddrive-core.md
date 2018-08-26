# dDrive Core

## Introduction
dDrive is a secure, real time, distributed disk and file system.

``` js
npm install @ddrive/core
```

## Usage

dDrive aims to implement the same API as Node.js' core FS module.

``` js
var ddrive = require('@ddrive/core')
var vault = ddrive('./ddrive') // content will be stored in this folder

vault.writeFile('/greetings.txt', 'martian', function (err) {
  if (err) throw err
  vault.readdir('/', function (err, list) {
    if (err) throw err
    console.log(list) // prints ['greetings.txt']
    vault.readFile('/greetings.txt', 'utf-8', function (err, data) {
      if (err) throw err
      console.log(data) // prints 'martian'
    })
  })
})
```

A big difference is that you can replicate the file system to other computers! All you need is a stream.

``` js
var net = require('net')

// ... on one machine

var server = net.createServer(function (socket) {
  socket.pipe(vault.replicate()).pipe(socket)
})

server.listen(10000)

// ... on another

var forkedVault = ddrive('./forked-ddrive', origKey)
var socket = net.connect(10000)

socket.pipe(forkedVault.replicate()).pipe(socket)
```

It also comes with build in versioning and real time replication. See more below.

## API

#### `var vault = ddrive(storage, [key], [options])`

Create a new ddrive.

The `storage` parameter defines how the contents of the vault will be stored. It can be one of the following, depending on how much control you require over how the vault is stored.

- If you pass in a string, the vault content will be stored in a folder at the given path.
- You can also pass in a function. This function will be called with the name of each of the required files for the vault, and needs to return a [`@dwcore/res`](https://github.com/dwcore/res/) instance.
- If you require complete control, you can also pass in an object containing a `metadata` and a `content` field. Both of these need to be functions, and are called with the following arguments:

  - `name`: the name of the file to be stored
  - `opts`
    - `key`: the [ddb key](http://docs.ddatabase.io/core#ddbkey) of the underlying dDatabase instance
    - `revelationKey`: the [revelation key](http://docs.ddatabase.io/core#ddb-revelation-key) of the underlying dDatabase instance
  - `vault`: the current DDrive instance

  The functions need to return a a [`@dwcore/res`](https://github.com/dwcore/res/) instance.

Options include:

``` js
{
  thin: true, // only download data on content ddb when it is specifically requested
  thinMetadata: true // only download data on metadata ddb when requested
  metadataStorageCacheSize: 65536 // how many entries to use in the metadata ddatabase's LRU cache
  contentStorageCacheSize: 65536 // how many entries to use in the content ddatabase's LRU cache
  treeCacheSize: 65536 // how many entries to use in the append-tree's LRU cache
}
```

Note that a forked dDrive vault can be "thin". Usually (by setting `thin: true`) this means that the content is not downloaded until you ask for it, but the entire metadata ddb is still downloaded. If you want a _very_ thin vault, where even the metadata ddb is not downloaded until you request it, then you should _also_ set `thinMetadata: true`.

#### `var stream = vault.replicate([options])`

Replicate this vault. Options include

``` js
{
  live: false, // keep replicating
  download: true, // download data from peers?
  upload: true // upload data to peers?
}
```

#### `vault.version`

Get the current version of the vault (incrementing number).

#### `vault.key`

The public key identifying the vault.

#### `vault.revelationKey`

A key derived from the public key that can be used to revelation other peers sharing this vault.

#### `vault.writable`

A boolean indicating whether the vault is writable.

#### `vault.on('ready')`

Emitted when the vault is fully ready and all properties has been populated.

#### `vault.on('error', err)`

Emitted when a critical error during load happened.

#### `var oldDrive = vault.checkout(version, [opts])`

Checkout a readonly copy of the vault at an old version. Options are used to configure the `oldDrive`:

```js
{
  metadataStorageCacheSize: 65536 // how many entries to use in the metadata ddatabase's LRU cache
  contentStorageCacheSize: 65536 // how many entries to use in the content ddatabase's LRU cache
  treeCacheSize: 65536 // how many entries to use in the append-tree's LRU cache
}
```

#### `vault.download([path], [callback])`

Download all files in path of current version.
If no path is specified this will download all files.

You can use this with `.checkout(version)` to download a specific version of the vault.

``` js
vault.checkout(version).download()
```

#### `var stream = vault.history([options])`

Get a stream of all changes and their versions from this vault.

#### `var stream = vault.createReadStream(name, [options])`

Read a file out as a stream. Similar to fs.createReadStream.

Options include:

``` js
{
  start: optionalByteOffset, // similar to fs
  end: optionalInclusiveByteEndOffset, // similar to fs
  length: optionalByteLength
}
```

#### `vault.readFile(name, [options], callback)`

Read an entire file into memory. Similar to fs.readFile.

Options can either be an object or a string

Options include:
```js
{
  encoding: string
  cached: true|false // default: false
}
```
or a string can be passed as options to simply set the encoding - similar to fs.

If `cached` is set to `true`, this function returns results only if they have already been downloaded.

#### `var stream = vault.createDiffStream(version, [options])`

Diff this vault this another version. `version` can both be a version number of a checkout instance of the vault. The `data` objects looks like this

``` js
{
  type: 'put' | 'del',
  name: '/some/path/name.txt',
  value: {
    // the stat object
  }
}
```

#### `var stream = vault.createWriteStream(name, [options])`

Write a file as a stream. Similar to fs.createWriteStream.
If `options.cached` is set to `true`, this function returns results only if they have already been downloaded.

#### `vault.writeFile(name, buffer, [options], [callback])`

Write a file from a single buffer. Similar to fs.writeFile.

#### `vault.unlink(name, [callback])`

Unlinks (deletes) a file. Similar to fs.unlink.

#### `vault.mkdir(name, [options], [callback])`

Explictly create an directory. Similar to fs.mkdir

#### `vault.rmdir(name, [callback])`

Delete an empty directory. Similar to fs.rmdir.

#### `vault.readdir(name, [options], [callback])`

Lists a directory. Similar to fs.readdir.

Options include:

``` js
{
    cached: true|false, // default: false
}
```

If `cached` is set to `true`, this function returns results from the local version of the vault’s append-tree. Default behavior is to fetch the latest remote version of the vault before returning list of directories.

#### `vault.stat(name, [options], callback)`

Stat an entry. Similar to fs.stat. Sample output:

```
Stat {
  dev: 0,
  nlink: 1,
  rdev: 0,
  blksize: 0,
  ino: 0,
  mode: 16877,
  uid: 0,
  gid: 0,
  size: 0,
  offset: 0,
  blocks: 0,
  atime: 2017-04-10T18:59:00.147Z,
  mtime: 2017-04-10T18:59:00.147Z,
  ctime: 2017-04-10T18:59:00.147Z,
  linkname: undefined }
```

The output object includes methods similar to fs.stat:

``` js
var stat = vault.stat('/greetings.txt')
stat.isDirectory()
stat.isFile()
```

Options include:
```js
{
  cached: true|false // default: false,
  wait: true|false // default: true
}
```

If `cached` is set to `true`, this function returns results only if they have already been downloaded.

If `wait` is set to `true`, this function will wait for data to be downloaded. If false, will return an error.

#### `vault.lstat(name, [options], callback)`

Stat an entry but do not follow symlinks. Similar to fs.lstat.

Options include:
```js
{
  cached: true|false // default: false,
  wait: true|false // default: true
}
```

If `cached` is set to `true`, this function returns results only if they have already been downloaded.

If `wait` is set to `true`, this function will wait for data to be downloaded. If false, will return an error.

#### `vault.access(name, [options], callback)`

Similar to fs.access.

Options include:
```js
{
  cached: true|false // default: false,
  wait: true|false // default: true
}
```

If `cached` is set to `true`, this function returns results only if they have already been downloaded.

If `wait` is set to `true`, this function will wait for data to be downloaded. If false, will return an error.

#### `vault.open(name, flags, [mode], callback)`

Open a file and get a file descriptor back. Similar to fs.open.

Note that currently only read mode is supported in this API.

#### `vault.read(fd, buf, offset, len, position, callback)`

Read from a file descriptor into a buffer. Similar to fs.read.

#### `vault.close(fd, [callback])`

Close a file. Similar to fs.close.

#### `vault.close([callback])`

Closes all open resources used by the vault.
The vault should no longer be used after calling this.

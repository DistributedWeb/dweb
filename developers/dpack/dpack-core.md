# dPack Core Javascript Library

## Features

* High-level glue for common dPack and [dDrive](https://docs.ddrive.io) modules.
* Sane defaults and consistent management of storage & secret keys across applications, using [@dpack/storage](/storage).
* Easily connect to the dPack network, using [flock-revelation](https://github.com/distributedweb/flock-revelation)
* Import files from the file system, using [dPack Mirror](/mirror)
* Serve dPacks over http with [http](/cli/http)
* Access APIs to lower level modules with a single `require`!

## dApp Support

Many of our dependencies work in the browser, but `dpack-core` is tailored for file system applications. See [dpackJS](/dpackjs) if you want to build browser-friendly dWeb dApps.

## Example

To send files via dPack:

1. Tell `dpack-core` where the files are.
2. Import the files.
3. Share the files on the dWeb network! (And share the link)

```js
var DWeb = require('@dpack/core')

// 1. My files are in /dpack-example/cancer-analysis
DWeb('/dpack-example/cancer-analysis', function (err, dweb) {
  if (err) throw err

  // 2. Import the files
  dweb.importFiles()

  // 3. Share the files on the network!
  dweb.joinNetwork()
  // (And share the link)
  console.log('My dPack link is: dweb://', dweb.key.toString('hex'))
})
```

These files are now available to share over the dWeb network via the key printed in the console.

To download the files, you can make another `dpack-core` instance in a different folder. This time we also have three steps:

1. Tell dPack where I want to download the files.
2. Tell dPack what the link is.
3. Join the dWeb network and download the dPack!

```js
var DWeb = require('@dpack/core')

// 1. Tell DWeb where to download the files
DWeb('/download/cat-analysis', {
  // 2. Tell DWeb what link I want
  key: '<dweb-key>' // (a 64 character hash from above)
}, function (err, dweb) {
  if (err) throw err

  // 3. Join the network & download (files are automatically downloaded)
  dweb.joinNetwork()
})
```

Thats it! By default, all files are automatically downloaded when you connect to the other users.


## Usage

All dpack-core applications have a similar structure around three main elements:

1. **Storage** - where the files and metadata are stored.
2. **Network** - connecting to other users to upload or download data.
3. **Adding Files** - adding files from the file system to the dDrive vault.

We'll go through what these are for and a few of the common usages of each element.

### Storage

Every dPack vault has **storage**, this is the required first argument for dpack-core. By default, we use [@dpack/storage](/storage) which stores the secret key in `~/.dweb/` and the rest of the ddata in `dir/.dweb`. Other common options are:

* **Persistent storage**: Stored files in `/dpack-dir` and metadata in `dpack-dir/.dweb` by passing `/dpack-dir` as the first argument.
* **Temporary Storage**: Use the `temp: true` option to keep metadata stored in memory.

```js
// Permanent Storage
DWeb('/dpack-dir', function (err, dweb) {
  // Do DWeb Stuff
})

// Temporary Storage
DWeb('/dpack-dir', {temp: true}, function (err, dweb) {
  // Do DWeb Stuff
})
```

Both of these will import files from `/dpack-dir` when doing `dweb.importFiles()` but only the first will make a `.dweb` folder and keep the metadata on disk.

The storage argument can also be passed through to dDrive for more advanced storage use cases.

### Network

DWeb is all about the network! You'll almost always want to join the network right after you create your DWeb:

```js
DWeb('/dpack-dir', function (err, dweb) {
  dweb.joinNetwork()
  dweb.network.on('connection', function () {
    console.log('I connected to someone!')
  })
})
```

#### Downloading Files

Remember, if you are downloading - metadata and file downloads will happen automatically once you join the network!

dPack runs on a peer to peer network, sometimes there may not be anyone online for a particular key. You can make your application more user friendly by using the callback in `joinNetwork`:

```js
// Downloading <key> with joinNetwork callback
DWeb('/dpack-dir', {key: '<key>'}, function (err, dweb) {
  dweb.joinNetwork(function (err) {
    if (err) throw err

    // After the first round of network checks, the callback is called
    // If no one is online, you can exit and let the user know.
    if (!dweb.network.connected || !dweb.network.connecting) {
      console.error('No users currently online for that key.')
      process.exit(1)
    }
  })
})
```

##### Download on Demand

If you want to control what files and metadata` (dweb.json)` are downloaded, you can use the thin option:

```js
// Downloading <key> with thin option
DWeb('/dpack-dir', {key: '<key>', thin: true}, function (err, dweb) {
  dweb.joinNetwork()

  // Manually download files via the dDrive API:
  dweb.vault.readFile('/cat-locations.txt', function (err, content) {
    console.log(content) // prints cat-locations.txt file!
  })
})
```

dPack will only download metadata and content for the parts you request with `thin` mode!

### Importing Files

There are many ways to get files imported into an vault! dPack node provides a few basic methods. If you need more advanced imports, you can use the `vault.createWriteStream()` methods directly.

By default, just call `dweb.importFiles()` to import from the directory you initialized with. You can watch that folder for changes by setting the watch option:

```js
DWeb('/my-data', function (err, dweb) {
  if (err) throw err

  var progress = dweb.importFiles({watch: true}) // with watch: true, there is no callback
  progress.on('put', function (src, dest) {
    console.log('Importing ', src.name, ' into vault')
  })
})
```

You can also import from another directory:

```js
DWeb('/my-data', function (err, dweb) {
  if (err) throw err

  dweb.importFiles('/another-dir', function (err) {
    console.log('done importing another-dir')
  })
})
```

That covers some of the common use cases, let us know if there are more to add! Keep reading for the full API docs.

## API

### `DWeb(dir|storage, [opts], callback(err, dweb))`

Initialize a dPack Vault in `dir`. If there is an existing dPack Vault, the vault will be resumed.

#### Storage

* `dir` (Default) - Use [@dpack/storage](/storage) inside `dir`. This stores files as files, sleep files inside `.dweb`, and the secret key in the user's home directory.
* `dir` with `opts.latest: false` - Store as SLEEP files, including storing the content as a `content.data` file. This is useful for storing all history in a single flat file.
* `dir` with `opts.temp: true` - Store everything in memory (including files).
* `storage` function - pass a custom storage function along to dDrive, see `@dpack/storage` for an example.

Most options are passed directly to the module you're using (e.g. `dweb.importFiles(opts)`. However, there are also some initial `opts` can include:

```js
opts = {
  key: '<dweb-key>', // existing key to create vault with or resume
  temp: false, // Use dwrem as the storage.

  // dDrive options
  thin: false // download only files you request
}
```

The callback, `cb(err, dweb)`, includes a `dweb` object that has the following properties:

* `dweb.key`: key of the dPack (this will be set later for non-live vaults)
* `dweb.vault`: dDrive vault instance.
* `dweb.path`: Path of the dPack Vault
* `dweb.live`: `vault.live`
* `dweb.writable`: Is the `vault` writable?
* `dweb.resumed`: `true` if the vault was resumed from an existing database
* `dweb.options`: All options passed to dPack and the other submodules

### Module Interfaces

**`dpack-core` provides an easy interface to common dPack modules for the created dWeb Vault on the `dWeb` object provided in the callback:**

#### `var network = dweb.joinNetwork([opts], [cb])`

Join the network to start transferring data for `dweb.key`, using [flock-revelation](https://github.com/distributedweb/flock-revelation). You can also use `dweb.join([opts], [cb])`.

If you specify `cb`, it will be called *when the first round* of revelation has completed. This is helpful to check immediately if peers are available and if not fail gracefully, more similar to http requests.

Returns a `network` object with properties:

* `network.connected` - number of peers connected
* `network.on('listening')` - emitted with network is listening
* `network.on('connection', connection, info)` - Emitted when you connect to another peer. Info is an object that contains info about the connection

##### Network Options

`opts` are passed to flock-revelation, which can include:

```js
opts = {
  upload: true, // announce and upload data to other peers
  download: true, // download data from other peers
  port: 3282, // port for revelation flock
  utp: true, // use utp in revelation flock
  tcp: true // use tcp in revelation flock
}

//Defaults from @dweb/flock-presets can also be overwritten:

opts = {
  dns: {
    server: // DNS server
    domain: // DNS domain
  }
  dht: {
    bootstrap: // distributed hash table bootstrapping nodes
  }
}
```

Returns a [flock-revelation](https://github.com/distributedweb/flock-revelation) instance.

#### `dweb.leaveNetwork()` or `dweb.leave()`

Leaves the network for the vault.

#### `var importer = dweb.importFiles([src], [opts], [cb])`

**Vault must be writable to import.**

Import files to your dPack Vault from the directory using [dPack Mirror](/mirror).

* `src` - By default, files will be imported from the folder where the vault was initiated. Import files from another directory by specifying `src`.
* `opts` - options passed to mirror-folder (see below).
* `cb` - called when import is finished.

Returns a `importer` object with properties:

* `importer.on('error', err)`
* `importer.on('put', src, dest)` - file put started. `src.live` is true is file was added by file watch event.
* `importer.on('put-data', chunk)` - chunk of file added
* `importer.on('put-end', src, dest)` - end of file write stream
* `importer.on('del', dest)` - file deleted from dest
* `importer.on('end')` - Emits when mirror is done (not emitted in watch mode)
* If `opts.count` is true:
  * `importer.on('count', {files, bytes})` - Emitted after initial scan of src directory. See import progress section for details.
  * `importer.count` will be `{files, bytes}` to import after initial scan.
  * `importer.putDone` will track `{files, bytes}` for imported files.

##### Importer Options

Options include:

```js
var opts = {
  count: true, // do an initial dry run import for rendering progress
  ignoreHidden: true, // ignore hidden files  (if false, .dweb will still be ignored)
  ignoreDirs: true, // do not import directories (dDrive does not need them and it pollutes metadata)
  useDWebIgnore: true, // ignore entries in the `.dwebignore` file from import dir target.
  ignore: // (see below for default info) anymatch expression to ignore files
  watch: false, // watch files for changes & import on change (vault must be live)
}
```

##### Ignoring Files

You can use a `.dwebignore` file in the imported directory, `src`, to ignore any the user specifies. This is done by default.

`dpack-core` uses [@dpack/ignore](/ignore) to provide a default ignore option, ignoring the `.dweb` folder and all hidden files or directories. Use `opts.ignoreHidden = false` to import hidden files or folders, except the `.dweb` directory.

*It's important that the `.dweb` folder is not imported because it contains a private key that allows the owner to write to the vault.*

#### `var stats = dweb.trackStats()`

##### `stats.on('update')`

Emitted when vault stats are updated. Get new stats with `stats.get()`.

##### `var st = dweb.stats.get()`

`dweb.trackStats()` adds a `stats` object to `dweb`.  Get general vault stats for the latest version:

```js
{
  files: 12,
  byteLength: 1234,
  length: 4, // number of blocks for latest files
  version: 6, // vault.version for these stats
  downloaded: 4 // number of downloaded blocks for latest
}
```

##### `stats.network`

Get upload and download speeds: `stats.network.uploadSpeed` or `stats.network.downloadSpeed`. Transfer speeds are tracked using [@ddrive/netspeed](https://docs.ddrive.io/netspeed/).

##### `var peers = stats.peers`

* `peers.total` - total number of connected peers
* `peers.complete` - connected peers with all the content data

#### `var server = dweb.serveHttp(opts)`

Serve files over http via [@ddrive/http](https://docs.ddrive.io/http). Returns a node http server instance.

```js
opts = {
  port: 8080, // http port
  live: true, // live update directory index listing
  footer: 'Served via DWeb.', // Set a footer for the index listing
  exposeHeaders: false // expose dPack key in headers
}
```

#### `dweb.pause()`

Pause all upload & downloads. Currently, this is the same as `dweb.leaveNetwork()`, which leaves the network and destroys the flock. Revelation will happen again on `resume()`.

#### `dweb.resume()`

Resume network activity. Current, this is the same as `dweb.joinNetwork()`.

#### `dweb.close(cb)`

Stops replication and closes all the things opened for `dpack-core`, including:

* `dweb.vault.close(cb)`
* `dweb.network.close(cb)`
* `dweb.importer.destroy()` (file watcher)

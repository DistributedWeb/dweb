# dAppDB

## Introduction

Distributed scalable database for the distributed web protocol (dweb://).

```
npm config set @dappdb:registry http://npm.dwebs.io
npm install @dappdb/core
```

# DappDB Architecture

DappDB is a scalable peer-to-peer key-value database for the distributed web protocol (dweb://) and other distributed web protocols.

## Filesystem metaphor

DappDB is structured to be used much like a traditional hierarchical
filesystem. A value can be written and read at locations like `/foo/bar/baz`,
and the API supports querying or tracking values at subpaths, like how watching
for changes on `/foo/bar` will report both changes to `/foo/bar/baz` and also
`/foo/bar/19`.

## Set of append-only logs (feeds)

A DappDB is fundamentally a set of
[@ddatabase/core](https://github.com/ddatabase/core)s. A *@ddatabase/core* is a secure
append-only log that is identified by a public key, and can only be written to
by the holder of the corresponding private key. Because it is append-only, old
values cannot be deleted nor modified. Because it is secure, a feed can be
downloaded from even untrustworthy peers and verified to be accurate. Any
modifications (malicious or otherwise) to the original feed data by someone
other than the author can be readily detected.

Each entry in a @ddatabase/core has a *sequence number*, that increments by 1 with
each write, starting at 0 (`seq=0`).

DappDB builds its hierarchical key-value store on top of these @ddatabase/core feeds,
and also provides facilities for authorization, and replication of those member
@ddatabase/cores.

### Directed acyclic graph

The combination of all operations performed on a DappDB by all of its members
forms a DAG (*directed acyclic graph*). Each write to the database (setting a
key to a value) includes information to point backward at all of the known
"heads" in the graph.

To illustrate what this means, let's say Alice starts a new DappDB and writes 2
values to it:

```
// Feed

0 (/foo/bar = 'baz')
1 (/foo/2   = '{ "some": "json" }')


// Graph

Alice:  0  <---  1
```

Where sequence number 1 (the second entry) refers to sequence number 0 on the
same feed (Alice's).

Now Alice *authorizes* Bob to write to the DappDB. Internally, this means Alice
writes a special message to her feed saying that Bob's feed (identified by his
public key) should be read and replicated in by other participants. Her feed
becomes

```
// Feed

0 (/foo/bar = 'baz')
1 (/foo/2   = '{ "some": "json" }')
2 (''       = '')


// Graph

Alice: 0  <---  1  <---  2
```

Authorization is formatted internally in a special way so that it isn't
interpreted as a key/value pair.

Now Bob writes a value to his feed, and then Alice and Bob sync. The result is:

```
// Feed

//// Alice
0 (/foo/bar = 'baz')
1 (/foo/2   = '{ "some": "json" }')
2 (''       = '')

//// Bob
0 (/a/b     = '12')


// Graph

Alice: 0  <---  1  <---  2
Bob  : 0
```

Notice that none of Alice's entries refer to Bob's, and vice versa. This is
because neither has written any entries to their feeds since the two became
aware of each other (authorized & replicated each other's feeds).

Right now there are two "heads" of the graph: Alice's feed at seq 2, and Bob's
feed at seq 0.

Next, Alice writes a new value, and her latest entry will refer to Bob's:

```
// Feed

//// Alice
0 (/foo/bar = 'baz')
1 (/foo/2   = '{ "some": "json" }')
2 (''       = '')
3 (/foo/hup = 'beep')

//// Bob
0 (/a/b     = '12')


// Graph

Alice: 0  <---  1  <---  2  <--/  3
Bob  : 0  <-------------------/
```

Because Alice's latest feed entry refers to Bob's latest feed entry, there is
now only one "head" in the database. That means there is enough information in
Alice's seq=3 entry to find any other key in the database. In the last example,
there were two heads (Alice's seq=2 and Bob's seq=0); both of which would need
to be read internally in order to locate any key in the database.

Now there is only one "head": Alice's feed at seq 3.

## Authorization

The set of @ddatabase/cores are *authorized* in that the original author of the first
@ddatabase/core in a hyperdb must explicitly denote in their append-only log that the
public key of a new @ddatabase/core is permitted to edit the database. Any authorized
member may authorize more members. There is no revocation or other author
management elements currently.

## Incremental index

DappDB builds an *incremental index* with every new key/value pairs ("nodes")
written. This means a separate data structure doesn't need to be maintained
elsewhere for fast writes and lookups: each node written has enough information
to look up any other key quickly and otherwise navigate the database.

Each node stores the following basic information:

- `key`: the key that is being created or modified. e.g. `/home/sww/dev.md`
- `value`: the value stored at that key.
- `seq`: the sequence number of this entry in the owner's @ddatabase/core. 0 is the
  first, 1 the second, and so forth.
- `feed`: the ID of the @ddatabase/core writer that wrote this
- `path`: a 2-bit hash sequence of the key's components
- `trie`: a navigation structure used with `path` to find a desired key
- `clock`: vector clock to determine node insertion causality
- `feeds`: an array of { feedKey, seq } for decoding a `clock`

### Vector clock

Each node stores a [vector clock](https://en.wikipedia.org/wiki/Vector_clock) of
the last known sequence number from each feed it knows about. This is what forms
the DAG structure.

A vector clock on a node of, say, `[0, 2, 5]` means:

- when this node was written, the largest seq # in my local fed is 0
- when this node was written, the largest seq # in the second feed I have is 2
- when this node was written, the largest seq # in the third feed I have is 5

For example, Bob's vector clock for Alice's seq=3 entry above would be `[0, 3]`
since he knows of her latest entry (seq=3) and his own (seq=0).

The vector clock is used for correctly traversing history. This is necessary for
the `db#heads` API as well as `db#createHistoryStream`.

### Prefix trie

Given a DappDB with hundreds of entries, how can a key like `/a/b/c` be looked
up quickly?

Each node stores a *prefix [trie](https://en.wikipedia.org/wiki/Trie)* that
assists with finding the shortest path to the desired key.

When a node is written, its *prefix hash* is computed. This done by first
splitting the key into its components (`a`, `b`, and `c` for `/a/b/c`), and then
hashing each component into a 32-character hash, where one character is a 2-bit
value (0, 1, 2, or 3). The `prefix` hash for `/a/b/c` is

```js
node.path = [
1, 2, 0, 1, 2, 0, 2, 2, 3, 0, 1, 2, 1, 3, 0, 3, 0, 0, 2, 1, 0, 2, 0, 0, 2, 0, 0, 3, 2, 1, 1, 2,
0, 1, 2, 3, 2, 2, 2, 0, 3, 1, 1, 3, 0, 3, 1, 3, 0, 1, 0, 1, 3, 2, 0, 2, 2, 3, 2, 2, 3, 3, 2, 3,
0, 1, 1, 0, 1, 2, 3, 2, 2, 2, 0, 0, 3, 1, 2, 1, 3, 3, 3, 3, 3, 3, 0, 3, 3, 2, 3, 2, 3, 0, 1, 0,
4 ]
```

Each component is divided by a newline. `4` is a special value indicating the
end of the prefix.

#### Example

Consider a fresh DappDB. We write `/a/b = 24` and get back this node:

```js
{ key: '/a/b',
  value: '24',
  clock: [ 0 ],
  trie: [],
  feeds: [ [Object] ],
  feedSeq: 0,
  feed: 0,
  seq: 0,
  path:
   [ 1, 2, 0, 1, 2, 0, 2, 2, 3, 0, 1, 2, 1, 3, 0, 3, 0, 0, 2, 1, 0, 2, 0, 0, 2, 0, 0, 3, 2, 1, 1, 2,
     0, 1, 2, 3, 2, 2, 2, 0, 3, 1, 1, 3, 0, 3, 1, 3, 0, 1, 0, 1, 3, 2, 0, 2, 2, 3, 2, 2, 3, 3, 2, 3,
     4 ] }
```

If you compare this path to the one for `/a/b/c` above, you'll see that the
first 64 2-bit characters match. This is because `/a/b` is a prefix of `/a/b/c`.

Since this is the first entry, `seq` is 0. Since this is the only known feed,
`feed` is also 0. `feeds` is an array of entries of the form `{ key: Buffer,
seq: Number }` that let you map the numeric value `feed` to a @ddatabase/core key and
its sequence number head. `feeds` isn't always set: it only gets included when
it changes compared to `node.seq - 1`, in the interest of storing less data per
node.

Now we write `/a/c = hello` and get this node:

```js
{ key: '/a/c',
  value: 'hello',
  clock: [ 0 ],
  trie: [ , , , , , , , , , , , , , , , , , , , , , , , , , , , , , , , , , , [ , , [ { feed: 0, seq: 0 } ] ] ],
  feeds: [],
  feedSeq: 0,
  feed: 0,
  seq: 1,
  path:
   [ 1, 2, 0, 1, 2, 0, 2, 2, 3, 0, 1, 2, 1, 3, 0, 3, 0, 0, 2, 1, 0, 2, 0, 0, 2, 0, 0, 3, 2, 1, 1, 2,
     0, 1, 1, 0, 1, 2, 3, 2, 2, 2, 0, 0, 3, 1, 2, 1, 3, 3, 3, 3, 3, 3, 0, 3, 3, 2, 3, 2, 3, 0, 1, 0,
     4 ] }
```

As expected, this node has the same `feed` value as before (since we're only
writing to one feed). Its `seq` is 1, since the last was 0. Notice that `feeds`
isn't included, because the mapping of the numeric `feed` value to a key hasn't
changed.

Also, this and the previous node have the first 32 characters of their `path` in
common (the prefix `/a`).

Notice though that `trie` is set. It's a long but sparse array. It has 35
entries, with the last one referencing the first node inserted (`a/b/`). Why?

(If it wasn't stored as a sparse array, you'd actually see 64 entries (the
length of the `path`). But since the other 29 entries are also empty, hyperdb
doesn't bother allocating them.)

If you visually compare this node's `path` with the previous node's `path`, how
many entries do they have in common? At which entry do the 2-bit numbers
diverge?

At the 35th entry.

What this is saying is "if the hash of the key you're looking for differs from
mine on the 35th entry, you want to travel to `{ feed: 0, seq: 0 }` to find the
node you're looking for.

This is how finding a node works, starting at any other node:

1. Compute the 2-bit hash sequence of the key you're after (e.g. `a/b`)
2. Lookup the newest entry in the feed.
3. Compare its `path` against the hash you just computed.
4. If you discover that the `path` and your hash match, then this is the node
   you're looking for!
5. Otherwise, once a 2-bit character from `path` and your hash disagree, note
   the index # where they differ and look up that value in the node's `trie`.
   Fetch that node at the given feed and sequence number, and go back to step 3.
   Repeat until you reach step 4 (match) or there is no entry in the node's trie
   for the key you're after (no match).

What if there are multiple feeds in the DappDB? The lookup algorithm changes
slightly. Replace the above step 2 for:

> 2. Fetch the latest entry from *every* feed. For each head node, proceed to
>    the next step.

The other steps are the same as before.


## Usage

``` js
var dappdb = require('@dappdb/core')

var db = dappdb('./my.db', {valueEncoding: 'utf-8'})

db.put('/hello', 'world', function (err) {
  if (err) throw err
  db.get('/hello', function (err, nodes) {
    if (err) throw err
    console.log('/hello --> ' + nodes[0].value)
  })
})
```

## API

#### `var db = dappdb(storage, [key], [options])`

Create a new dAppDB.

`storage` can be a string or a function. If a string like the above example, the
[random-access-file](https://github.com/mafintosh/random-access-file) storage
module is used; the resulting folder with the data will be whatever `storage` is
set to.

If `storage` is a function, it will be called with every filename dAppDB needs
to operate on. There are many providers for the
[abstract-random-access](https://github.com/juliangruber/abstract-random-access)
interface. e.g.

```js
var ram = require('random-access-memory')
var feed = dappdb(function (filename) {
  // filename will be one of: data, bitfield, tree, signatures, key, secret_key
  // the data file will contain all your data concattenated.

  // just store all files in ram by returning a random-access-memory instance
  return ram()
})
```

`key` is a `Buffer` containing the local feed's public key. If you do not set
this the public key will be loaded from storage. If no key exists a new key pair
will be generated.

Options include:

```js
{
  map: node => mappedNode, // map nodes before returning them
  reduce: (a, b) => someNode, // reduce the nodes array before returning it
  firstNode: false, // set to true to reduce the nodes array to the first node in it
  valueEncoding: 'binary' // set the value encoding of the db
}
```

#### `db.key`

Buffer containing the public key identifying this dAppDB.

Populated after `ready` has been emitted. May be `null` before the event.

#### `db.revelationKey`

Buffer containing a key derived from the db.key.
In contrast to `db.key` this key does not allow you to verify the data but can be used to announce or look for peers that are sharing the same dAppDB, without leaking the dAppDB key.

Populated after `ready` has been emitted. May be `null` before the event.

#### `db.on('ready')`

Emitted exactly once: when the db is fully ready and all static properties have
been set. You do not need to wait for this when calling any async functions.

#### `db.version(callback)`

Get the current version identifier as a buffer for the db.

#### `var checkout = db.checkout(version)`

Checkout the db at an older version. The checkout is a DB instance as well.
Version should be a version identifier returned by the `db.version` api or an
array of nodes returned from `db.heads`.

#### `db.put(key, value, [callback])`

Insert a new value. Will merge any previous values seen for this key.

#### `db.get(key, callback)`

Lookup a string `key`. Returns a nodes array with the current values for this key.
If there is no current conflicts for this key the array will only contain a single node.

#### `db.del(key, callback)`

Delete a string `key`.

#### `db.batch(batch, [callback])`

Insert a batch of values efficiently, in a single atomic transaction. A batch should be an array of objects that look like this:

``` js
{
  type: 'put',
  key: someKey,
  value: someValue
}
```

`callback`'s parameters are `err, nodes`, where `nodes` is an array of the batched nodes.

#### `db.local`

Your local writable feed. You have to get an owner of the dAppDB to authorize you to have your
writes replicate. The first person to create the dAppDB is the first owner.

#### `db.authorize(key, [callback])`

Authorize another peer to write to the dAppDB.

To get another peer to authorize you you'd usually do something like

``` js
myDb.on('ready', function () {
  console.log('You local key is ' + myDb.local.key.toString('hex'))
  console.log('Tell an owner to authorize it')
})
```

#### `db.authorized(key, [callback])`

Check whether a key is authorized to write to the database.

```js
myDb.authorized(otherDb.local.key, function (err, auth) {
  if (err) console.log('err', err)
  else if (auth === true) console.log('authorized')
  else console.log('not authorized')
})
```

#### `watcher = db.watch(folderOrKey, onchange)`

Watch a folder and get notified anytime a key inside this folder
has changed.

```js
db.watch('foo/bar', function () {
  console.log('folder has changed')
})

...

db.put('foo/bar/baz', 'hi') // triggers the above
```

You can destroy the watcher by calling `watcher.destroy()`.

The watcher will emit `watching` when it starts watching and `change`
when a change has been detected.

If a critical error occurs an error will be emitted on the watcher.

#### `var stream = db.createReadStream(prefix[, options])`

Create a readable stream of nodes stored in the database.
Set `prefix` to only iterate nodes prefixed with that folder.

Options include:

```js
{
  recursive: true // visit all subfolders.
                  // set to false to only visit the first node in each folder
  reverse: true   // read the records in reverse order.
  gt: false       // visit only strictly nodes that are > than the prefix
}
```

#### `var stream = db.createWriteStream()`

Create a writable stream.

Where `stream.write(data)` accepts data as an object or an array of objects with the same form as `db.batch()`.

#### `db.list(prefix[, options], callback)`

Same as `createReadStream` but buffers the result to a list that is passed to the
callback.

#### `var stream = db.createDiffStream(prefix[, checkout)`

Find out about changes in key/value pairs between the version `checkout` and
current version prefixed by `prefix`.

`stream` is a readable object stream that outputs modifications like

```js
{ left: nodes, right: nodes }
```

`left` are the nodes for a key found in the `db` and `right` are the nodes found in the `checkout`.
If no nodes exist in the `db` for the key `left` will be `null` and vice versa.

#### `var stream = db.createHistoryStream([options])`

Returns a readable stream of node objects covering all historic values since the beginning of time.

Nodes are emitted in topographic order, meaning if value `v2` was aware of value
`v1` at its insertion time, `v1` must be emitted before `v2`.

To emit the nodes in reverse order pass `{reverse: true}` as an option.

#### `var stream = db.createKeyHistoryStream(key)`

Returns a readable stream of node objects covering all historic values for a specific key.

Results are returned with the latest value first.

#### `var stream = db.replicate([options])`

Create a replication stream. Options include:

``` js
{
  live: false // set to true to keep replicating
}
```

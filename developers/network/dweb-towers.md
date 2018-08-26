# dWeb Towers

## Introduction
dWeb Tower acts a beaming point, in the form of a server that is used by the overall network to coordinate handshaking with WebRTC and other protocols.

```
npm install @dwebs/tower
```

Or to install the command line tool

```
npm install @dwebs/tower
```

## dApp Usage

``` js
var dWebTower = require('@dwebs/tower')
var tower = dWebTower('@dapporg/dapp-name', [
  'http://your-tower.yourdomain.network'
])

hub.subscribe('your-flock-channel')
  .on('data', function (message) {
    console.log('new message received', message)
  })

hub.broadcast('your-flock-channel', {greetings: 'martian'})
```

## API

#### `tower = dWebTower(appName, urls)`

Create a new dWeb Tower client. If you have more than one hub running specify them in an array

``` js
// use more than one server for redundancy
var tower = dWebBeamTower('@dapporg/dapp-name', [
  'https://tower1.your-domain.network',
  'https://tower2.your-domain.network',
  'https://tower3.your-domain.network'
])
```

The `appName` is used to namespace the subscriptions/broadcast so you can reuse the
dWeb Tower for more than one app.

#### `stream = tower.subscribe(channel)`

Subscribe to a channel on the hub. Returns a readable stream of messages

#### `tower.broadcast(channel, message, [callback])`

Broadcast a new message to a channel on the hub

#### `tower.close([callback])`

Close all subscriptions

## CLI API

You can use the command line api to run a hub server

```
dwtower listen -p 8080 # starts a dWeb Tower server on 8080
```

To listen on https, use the `--key` and `--cert` flags to specify the path to the private
key and certificate files, respectively. These will be passed through to the node `https`
package.

To avoid logging to console on every subscribe/broadcast event use the `--quiet` or `-q` flag.

Or broadcast/subscribe to channels

```
dwtower broadcast my-app my-channel '{"greetings":"martian"}' -p 8080 -h tower1.your-domain.io
dwtower subscribe my-app my-channel -p 8080 -h tower1.your-domain.io
```

## Browser Compatibility

dWeb Tower can be used in any web browser, if you use a `browserify` build.

## Publicly available dWeb Towers

Through the magic of free hosting, here are some free open dWeb Tower servers!
For serious applications though, consider deploying your own dWeb Tower instances.

- http://tower1.dwebs.io (Netherlands)
- http://tower2.dwebs.io (Paris)
- http://tower3.dwebs.io (New York City)
- http://tower4.dwebs.io (San Francisco)
- http://tower5.dwebs.io (Dallas)
- http://tower6.dwebs.io (Singapore)

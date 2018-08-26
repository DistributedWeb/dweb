# dDatabase Daemon

## Introduction
A daemon that can download and share multiple dDatabases and [dDrives](http://docs.ddrive.io) with builtin in seeding support, over websockets.
Works directly with [@ddatabase/vaultr](/vaultr).

```
npm install -g @ddatabase/daemon
```

## Usage

``` js
ddbd
```

For more info run

```
Usage: ddbd [key?] [options]

    --cwd         [folder to run in]
    --websockets  [share over websockets as well]
    --port        [explicit websocket port]
    --no-flock    [disable flocking]
```

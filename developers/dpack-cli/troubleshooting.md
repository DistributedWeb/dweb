# dPack CLI Troubleshooting

## Troubleshooting Guide

### Installation Issues

#### Node & NPM

To use the dPack command line tool you will need to have [node and npm installed][https://docs.benos.io/utilities/installing-node].
Make sure those are installed correctly before installing dPack.
You can check the version of each:

```
node -v
npm -v
```

#### Global Install

The `-g` option installs dPack globally, allowing you to run it as a command.
Make sure you installed with that option.

* If you receive an `EACCES` error, it is a NPM permissions error. Try to use `sudo` in this event.
* If you receive an `EACCES` error, you may also install dPack with sudo: `sudo npm install -g @dpack/cli`.

### Debugging Output

If you are having trouble with a specific command, run with the debug environment variable set to `dweb` (and optionally also `dpack-core`).
This will help us debug any issues:

```
DEBUG=dpack,dpack-core dweb fork dweb://<link> dir
```

### Networking Issues

Networking capabilities vary widely with each computer, network, and configuration.
Whenever you run dPack there are several steps to distribute or download files with dWeb peers:

1. Discovering dWeb Peers
2. Connecting to dWeb Peers
3. Sending & Receiving Data

With successful use, dPack will show `Connected to 1 dWeb peer` after connection.
If you never see a dWeb peer connected, your network may be restricting revelation or connection.
Please try using the `dweb satoshi` command (see below) between the two computers not connecting. This will help troubleshoot dWeb's distributed network.

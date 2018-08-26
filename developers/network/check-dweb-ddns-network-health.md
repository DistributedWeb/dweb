# Checking dWeb dDNS Network Health

## dDNS Health Utility
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

## Dr. Satoshi dWeb Network Health Utility
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

# dPack Key Management and Moving dPacks

## Usage

`dweb keys` provides a few commands to help you move or backup your dPacks.

Writing to a dPack requires the secret key, stored in the `~/.dweb` folder. You can export and import these keys between dPacks. First, fork your dPacks to the new location:

* (original) `dweb dist`
* (duplicate) `dweb fork <link>`

Then transfer the secret key:

* (original) `dweb keys export` - copy the secret key printed out.
* (duplicate) `dweb keys import` - this will prompt you for the secret key, paste it in here.

# dPack CLI Documentation
---

Use dPack Command CLI to distribute files with version control, back up data to servers, browse remote files on demand, and automate long-term data preservation across the Distributed Web (dWeb).

## Table Of Contents
- [Installation](https://docs.dwebs.io/developers/dpack-cli/installation)
- [dPack CLI Help](#dpack-cli-menu)
- [dPack Command List](#dpack-command-list)
- [Usage](https://docs.dwebs.io/developers/dpack-cli/usage)
- [dPackJS](https://docs.dwebs.io/developers/dpack-cli/dpackjs)
- [Creating dPacks](https://docs.dwebs.io/developers/dpack-cli/creating-dpack)
- [Distributing dPacks](https://docs.dwebs.io/developers/dpack-cli/distributing-dpacks)
- [Downloading dPacks](https://docs.dwebs.io/developers/dpack-cli/downloading-dpacks)
- [Forking dPacks](https://docs.dwebs.io/developers/dpack-cli/forking-dpacks)
- [Key Management](https://docs.dwebs.io/developers/dpack-cli/key-management)
- [Selecting Files](https://docs.dwebs.io/developers/dpack-cli/selecting-files)
- [Shortcuts](https://docs.dwebs.io/developers/dpack-cli/shortcuts)
- [dPacks Over HTTP](https://docs.dwebs.io/developers/dpack-cli/http)
- [Syncing To The dWeb](https://docs.dwebs.io/developers/dpack-cli/sync)
- [Unpacking dPacks](https://docs.dwebs.io/developers/dpack-cli/unpacking-dpacks)
- [Dr. Satoshi](https://docs.dwebs.io/developers/dpack-cli/dr-satoshi)
- [dPack Repository](https://docs.dwebs.io/developers/dpack-cli/dpack-repository)
- [dPack Auth](https://docs.dwebs.io/developers/dpack-cli/dpack-auth)
- [Checking dPack Version](https://docs.dwebs.io/developers/dpack-cli/check-version)
- [dPack Quick Demos](#dpack-quick-demos)
- [Troubleshooting](https://docs.dwebs.io/developers/dpack-cli/troubleshooting)


### dPack CLI Menu

```
dweb help
```

### dPack Command List

```
$dweb

Usage: dweb <cmd> [<dir>] [options]

Create, Distribute and Sync Files:
   dweb create                  create empty dPack and dweb.json
   dweb dist                    create dPack, import files to the respository and distribute to the dWeb
   dweb sync                    import files to existing dPack & sync with network

Fork, Pull and Sync Files:
   dweb fork <link> [<dir>]     Fork a dPack via link to <dir>
   dweb pull                    Pull updates of a dPack & exit
   dweb sync                    Sync dPack with the dWeb

dPack Info:
   dweb log                     Print a dPack's history
   dweb status                  Prints key and information about a local dPack

dPack Developer Management:
   dweb <cmd> [<repository>]    All commands take <repository> option
   dweb register                Register new repository account
   dweb login                   Login to repository account
   dweb publish                 Publish dPack to repository
   dweb whoami                  Print repository account information
   dweb logout                  Logout from current repository

dPack Shortcut Commands:
   dweb <link> [<dir>]          Fork dPack or Sync dPack in <dir> with dWeb
   dweb <dir>                   Create dPack and sync dPack in <dir> with dWeb

dPack Troubleshooting & Help:
   dweb satoshi                 Check with Dr. Satoshi on the current health of the dWeb
   dweb help                    Print dPack manpage
   dweb <command> --help, -h    Print specific usage parameters for a dPack command
   dweb --version, -v           Print current dPack version


Have fun using dPack! Learn more at docs.dpack.io

```

### Quick Demos

To get started using dPack, you can try downloading a dPack and then distributing a dPack of your own.

This will download our demo files to the `~/Downloads/dpack-demo` folder. These files are being distributed by a server over dPack (to ensure high availability) but you may connect to any number of users also hosting the content.
You can also also view the files online: [dpacks.io/778f8d955175c92e4ced5e4f5563f69bfec0c86cc6f670352c457943666fe639](https://dpacks.io/778f8d955175c92e4ced5e4f5563f69bfec0c86cc6f670352c457943666fe639/). dpacks.io can download files over dPack and display them on HTTP as long as someone is hosting it. The website temporarily caches data for any visited links (do not view your dpack on dpacks.io if you do not want us to cache your data).


## Usage

The first time you run a command, a `.dweb` folder is created to store the dPack metadata (dweb.json).
Once a dPack is created, you can run all the commands inside that folder, similar to git.

dPack keeps secret keys in the `~/.dweb/secret_keys` folder. These are required to write to any dPacks you create.

# Downloading dPacks

## Usage
Start downloading by running the `fork` command. This creates a folder, downloads the content and metadata, and a `.dpack` folder inside. Once you started the download, you can resume using `fork` or the other download commands.

```
dweb fork <link> [<dir>] [--temp]
```

Clone a remote dPack vault to a local folder.
This will create a folder with the key name if no folder is specified.

### Downloading via `dweb.json` key

You can use a `dweb.json` file to fork also. This is useful when combining dPack and git, for example. To fork a dPack you can specify the path to a folder containing a `dweb.json`:

```
git clone git@github.com:distributedweb/dpack-git-example.git
dweb fork ./dpack-git-example
```

This will download the dPack specified in the `dweb.json` file.

### Download Demo

We made a demo folder just for this guide. Inside the demo folder is a `dweb.json` file and a gif. We distributed these files via the dWeb and now you can download them with our dPack key!

Similar to git, you can download somebody's dPack by running `dweb fork <link>`. You can also specify the directory:

```
❯ dweb fork dweb://7289f8d955175c92e4ced258297defec0c86cc235khsf8y82943666fe639 ~/Downloads/dweb-demo
dweb v1.0.1
Created new dweb in /Users/demo/Downloads/dweb-demo/.dweb
Forking: 2 files (1.4 MB)

2 connections | Download 614 KB/s Upload 0 B/s

dweb dweb complete.
Version 4
```

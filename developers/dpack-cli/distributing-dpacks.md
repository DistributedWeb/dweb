# Distributing dPacks

## Introduction

You can start distributing your files with a single command. Unlike `git`, you do not have to initialize a repository first, `dpack dist` will do that for you:

```
dweb dist <dir>
```

Use `dweb dist` to create a dPack and sync your files from your computer to other users. dPack scans your files inside `<dir>`, creating metadata in `<dir>/.dpack`. dPack stores the public link, version history, and file information inside the dPack Folder.


## Usage Example

The quickest way to get started distributing files is to use the `dist` command:

```
❯ dweb dist
dweb://3e830227b4b2be197679ff1b573cc85e689f202c0884eb8bdb0e1fcecbd93119
Distributing dPack: 24 files (383 MB)

0 connections | Download 0 B/s Upload 0 B/s

Importing 528 files to Archive (165 MB/s)
[=-----------------------------------------] 3%
ADD: data/expn_cd.csv (403 MB / 920 MB)
```


#### Distributing Demo

dPack can distribute files from your computer to anywhere. If you have a friend going through this demo with you, try distributing to them! If not we'll see what we can do. Find a folder on your computer to distribute. Inside the folder can be anything, dPack can handle all sorts of files (dPack works with really big folders too!).

First, you can create a new dPack inside that folder. Using the `dweb create` command also walks us through making a `dweb.json` file:

```
❯ dweb create
Welcome to dPack program!
You can turn any folder on your computer into a dPack.
A dPack is a folder with a dPack/dWeb configuration.
```

This will create a new (empty) dPack. dPack will print a link, distribute this link to give others access to view your files.

Once we have our dPack, run `dweb dist` to scan your files and sync them to the network. Distribute the link with your friend to instantly start downloading files.

You can also try viewing your files online. Go to [dpacks.io](https://dpacks.io) and enter your link to preview on the top left.

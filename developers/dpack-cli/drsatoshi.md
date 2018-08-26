# Dr. Satoshi Via dPack CLI
---

## Using Dr. Satoshi

We've included a tool to identify network issues with dPack.
Dr. Satoshi will run two tests:

1. Attempt to connect to a public server running a dWeb peer.
2. Attempt a direct connection between two peers. You will need to run the command on both the computers you are trying to distribute data between.

Start the satoshi by running:

```
dpack satoshi
```

For direct connection tests, Dr. Satoshi will print out a command to run on the other computer, `dpack satoshi <64-character-string>`.
Dr. Satoshi will run through the key steps in the process of distributing data between computers to help identify the issue.

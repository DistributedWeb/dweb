# dPack Repository Authentication
---

## Usage

You can use the `dweb` command line to register and publish to dPack repositories. dPack plans to support any repositories that are created or forked from our initial [dpacks.io](https://dpacks.io) distributed repository codebase. Currently, `dpacks.io` is the lone dWeb repository and the default for the dPack Command CLI.

To register and login you can use the following commands:

```
dweb register [<repository>]
dweb login
dweb whoami
```

Once you are logged in to a registry, you can publish a dPack vault:

```
cd my-repo
dweb create
dweb publish --name my-repo
```

- All repository requests take the `<repository>` option if you'd like to publish to a different repository than [dpacks.io](https://dpacks.io).
- You can deploy your own compatible [Repository server] if you'd rather use your own service.

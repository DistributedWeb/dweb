# Keeping dPacks Online Forever

## Table of contents

- [Installation (Ubuntu)](#installation-ubuntu)
- [Command Line Flags](#command-line-flags)
- [Env Vars](#env-vars)
- [Guides](#guides)
  - [Setup](#setup)
  - [Port setup](#port-setup)
  - [DNS records](#dns-records)
  - [Metrics Dashboard](#metrics-dashboard)
  - [Running dPack Forever behind Apache or Nginx](#running-forever-behind-apache-or-nginx)
- [Configuration File](#configuration-file)
  - [Directory](#directory)
  - [Domain](#domain)
  - [HttpMirror](#httpmirror)
  - [WebAPI](#webapi)
    - [webapi.username](#webapiusername)
    - [webapi.password](#webapipassword)
  - [LetsEncrypt](#letsencrypt)
    - [letsencrypt.email](#letsencryptemail)
    - [letsencrypt.agreeTos](#letsencryptagreetos)
  - [Ports](#ports)
    - [ports.http](#portshttp)
    - [ports.https](#portshttps)
  - [Dashboard](#dashboard)
    - [dashboard.port](#dashboardport)
  - [dPacks](#dwebs)
    - [dwebs.*.url](#dwebsurl)
    - [dwebs.*.name](#dwebsname)
    - [dwebs.*.otherDomains](#dwebsotherdomains)
  - [proxies](#proxies)
    - [proxies.*.from](#proxiesfrom)
    - [proxies.*.to](#proxiesto)
  - [redirects](#redirects)
    - [redirects.*.from](#redirectsfrom)
    - [redirects.*.to](#redirectsto)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Installation (Ubuntu)

You will need [nodejs](https://nodejs.org) version 8 or greater. (Consider using [nvm](https://nvm.sh) to get setup.)

You'll need to install some build dependencies:

```
# install build dependencies
sudo apt-get install libtool m4 automake libcap2-bin build-essential
```

Then install dPack Forever globally. See [this guide](https://docs.npmjs.com/getting-started/fixing-npm-permissions) if you run into permissions problems.

```
# install forever
npm install -g @dwebs/forever
```

Because dPack Forever will use privileged ports 80 and 443, you'll need to give nodejs permission to use them. (It's not recommended to run dPack Forever as super-user.)

```
# give node perms to use ports 80 and 443
sudo setcap cap_net_bind_service=+ep `readlink -f \`which node\``
```

If you want to run dPack Forever manually, you can invoke the command `forever`. However, for keeping the daemon running, we recommend [pm2](https://www.npmjs.com/package/pm2).

```
# install pm2
npm install -g pm2
```

Next, [setup your daemon](#setup).

## Command Line Flags

  - `--config <path>` use the config file at the given path instead of the default `~/.forever.yml`. Overrides the value of the `DPACK_FOREVER_CONFIG` env var.

## Env Vars

  - `DPACK_FOREVER_CONFIG=cfg_file_path` specify an alternative path to the config than `~/.forever.yml`
  - `NODE_ENV=debug|staging|production` set to `debug` or `staging` to use the lets-encrypt testing servers.

## Guides

### Setup

To configure your instance, edit `~/.forever.yml`. You can edit the configuration file even if the forever daemon is running, and forever will automatically restart after your changes are saved.

Here is an example config file:

```yaml
directory: ~/.forever # where your data will be stored
domain:                # enter your forever instance's domain here
httpMirror: true       # enables http mirrors of the dPacks
ports:
  http: 80             # HTTP port for redirects or non-SSL serving
  https: 443           # HTTPS port for serving mirrored content & DNS data
letsencrypt:           # set to false to disable lets-encrypt
  email:               # you must provide your email to LE for admin
  agreeTos: true       # you must agree to the LE terms (set to true)
dashboard:             # set to false to disable
  port: 8089           # port for accessing the metrics dashboard

# enable publishing to dPack Forever from dBrowser & dPack-CLI
webapi:                # set to false to disable
  username:            # the username for publishing from dBrowser/dPack-CLI
  password:            # the password for publishing from dBrowser/dPack-CLI

# enter your forever dPacks here
dwebs:
  - url:               # URL of the dPack to be kept online forever
    name:              # the name of the dPacks (sets the subdomain)
    otherDomains:      # (optional) the additional domains

# enter any proxied routes here
proxies:
  - from:              # the domain to accept requests from
    to:                # the domain (& port) to target

# enter any redirect routes here
redirects:
  - from:              # the domain to accept requests from
    to:                # the domain to redirect to
```

You'll want to configure the following items:

 - **Forever Domains**. Set the `domain:` field to the top-level domain name of your `dPack Forever` instance. New vaults will be hosted under its subdomains.
 - **dPack API**. Set a username and password on the Web API if you want to publish to your dPack Forever using `dBrowser` or the [dPack CLI](/cli).
 - **Let's Encrypt**. This is required for accessing your vaults with domain names. You'll need to provide your email address so that Let's Encrypt can warn you about expiring certs, or other issues. (Set this to `false` if you are running `dPack Forever` behind a proxy like Apache or Nginx.)
 - **dPacks**. Add the vaults that you want hosted. Each one will be kept online and made available at `dweb://{name}.yourdomain.com`. The `otherDomains` field is optional, and can take 1 or more additional domains for hosting the vault at. You can also add & remove vaults using dBrowser or the dPack CLI via the Web API.

Here's an example dPack with multiple domains. If the dPack Forever instance is hosted at `yourdomain.com`, then this dPack would be available at `dweb://mysite.yourdomain.com`, `dweb://mysite.com`, and `dweb://my-site.com`. (Don't forget to setup the DNS records!)

```yaml
dwebs:
  - url: dweb://1f968afe867f06b0d344c11efc23591c7f8c5fb3b4ac938d6000f330f6ee2a03/
    name: mysite
    otherDomains:
      - mysite.com
      - my-site.com
```

If your dPack Forever is running on ports 80/443, and you have other Web servers running on your server, you might need dPack Forever to proxy to those other servers. You can do that with the `proxies` config. Here's an example proxy rule:

```yaml
proxies:
  - from: my-proxy.com
    to: http://localhost:8080
```

Sometimes you need to redirect from old domains to new ones. You can do that with the `redirects` rule. Here's an example redirect rule:

```yaml
redirects:
  - from: my-old-site.com
    to: https://my-site.com
```

Now you're ready to start dPack Forever! If you want to run dPack Forever manually, you can invoke the command `forever`. However, for keeping the daemon running, we recommend [pm2](https://www.npmjs.com/package/pm2).

```
# start forever
pm2 start forever
```

To stop the daemon, run

```
# stop forever
pm2 stop forever
```

### Port setup

For dPack Forever to work correctly, you need to be able to access port 80 (http), 443 (https), and 3282 (dweb). Your firewall should be configured to allow traffic on those ports.

If you get an EACCES error on startup, you either have a process using the port already, or you lack permission to use the port. Try `lsof -i tcp:80` or `lsof -i tcp:443` to see if there are any processes bound to the ports you need.

If the ports are not in use, then it's probably a permissions problem. We recommend using the following command to solve that:

```
# give node perms to use ports 80 and 443
sudo setcap cap_net_bind_service=+ep `readlink -f \`which node\``
```

This will give nodejs the rights to use ports 80 and 443. This is preferable to running forever as root, because that carries some risk of a bug in forever allowing somebody to control your server.

### DNS records

You will need to create A records for all of the domains you use. For the subdomains, you can use a wildcard domain.

### Metrics Dashboard

dPack Forever has built-in support for [Prometheus](https://prometheus.io), which can be visualized by [Grafana](http://grafana.org/).

dPack Forever exposes its metrics at port 8089. Prometheus periodically scrapes the metrics, and stores them in a database. Grafana provides a nice dashboard. It's a little daunting at first, but setup should be relatively painless.

Follow these steps:

 1. [Install Prometheus](https://prometheus.io/download/) on your server.
 2. [Install Grafana](http://grafana.org/download/) on your server.
 3. Update the `prometheus.yml` config.
 4. Start prometheus and grafana.
 5. Login to grafana.
 6. Add prometheus as a data source to grafana. (It should be running at localhost:9090.)
 7. Import [this grafana dashboard](./grafana-dashboard.json).

Your prometheus.yml config should include have the scrape_configs set like this:

```yml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'forever'
    static_configs:
      - targets: ['localhost:8089']
```

### Running dPack Forever behind Apache or Nginx

If you are running dPack Forever on a server that uses Apache or Nginx, you may need to change your config to disable HTTPS. For instance, if you're using nginx and proxying to port `8080`, update your config to disable Let's Encrypt and to set the http port:

```yaml
letsencrypt: false
ports:
  http: 8080
```

You will need to add all domains to your Nginx/Apache config.

## Configuration file

The fields in detail.

### directory

The directory where forever will store your dPack vault's files. Defaults to ~/.forever.

### domain

The DNS domain of your forever instance.

### httpMirror

Set to `true` to provide https mirroring of your dPack vaults. Defaults to true.

### webapi

Set to `false` to disable the [dPack API](/api) which enables publishing to dPack Forever with [dBrowser](https://dbrowser.io) and the [dPack CLI](/cli). Defaults to `false`.

```yaml
# enable publishing to dPack Forever from dBrowser & dPack-CLI
webapi:                # set to false to disable
  username:            # the username for publishing from dBrowser/dPack-CLI
  password:            # the password for publishing from dBrowser/dPack-CLI
```

#### webapi.username

Sets the username for your pinning service API.

#### webapi.password

Sets the password for your pinning service API.

### letsencrypt

Set to `false` to disable Lets Encrypt's automatic SSL certificate provisioning. Defaults to `false`.

```yaml
letsencrypt:           # set to false to disable lets-encrypt
  email:               # you must provide your email to LE for admin
  agreeTos: true       # you must agree to the LE terms (set to true)
```

#### letsencrypt.email

The email to send Lets Encrypt notices to.

#### letsencrypt.agreeTos

Do you agree to the [terms of service of Lets Encrypt](https://letsencrypt.org/repository/)? (Required, must be true)

### ports

Contains the ports for http and https.

```yaml
ports:
  http: 80             # HTTP port for redirects or non-SSL serving
  https: 443           # HTTPS port for serving mirrored content & DNS data
```

#### ports.http

The port to serve the HTTP sites. Defaults to 80.

HTTP automatically redirects to HTTPS.

#### ports.https

The port to serve the HTTPS sites. Defaults to 443.

### dashboard

Set to `false` to disable the [prometheus metrics dashboard](#metrics-dashboard). Defaults to `false`.

```yaml
dashboard:             # set to false to disable
  port: 8089           # port for accessing the metrics dashboard
```

#### dashboard.port

The port to serve the [prometheus metrics dashboard](#metrics-dashboard). Defaults to 8089.

### dPacks

A listing of the dPack vaults to host.

You'll need to configure the DNS entry for the hostname to point to the server. For instance, if using `site.yourhostname.com`, you'll need a DNS entry pointing `site.yourhostname.com` to the server.

```yaml
dwebs:
  - url: dweb://1f968afe867f06b0d344c11efc23591c7f8c5fb3b4ac938d6000f330f6ee2a03/
    name: mysite
    otherDomains:
      - mysite.com
      - my-site.com
```

#### dwebs.*.url

The dPack URL of the site to host. Should be a 'raw' dPack dWeb-ready url (no DNS hostname). Example values:

```
868d6000f330f6967f06b3ee2a03811efc23591afe0d344cc7f8c5fb3b4ac91f
dweb://1f968afe867f06b0d344c11efc23591c7f8c5fb3b4ac938d6000f330f6ee2a03/
```

#### dwebs.*.name

The name of the dPack vault. Must be unique on the dPack Forever instance. The vault will be hosted at `{name}.yourhostname.com`. You'll need to configure the DNS entry for the hostname to point to the server.

#### dwebs.*.otherDomains

Additional domains of the dPack vault. Can be a string or a list of strings. Each string should be a domain name. Example values:

```
mysite.com
foo.bar.edu
best-site-ever.link
```

### proxies

A listing of domains to proxy. Useful when your server has other services running that you need available.

```yaml
proxies:
  - from: my-proxy.com
    to: http://localhost:8080
```

#### proxies.*.from

The domain to proxy from. Should be a domain name. Example values:

```
mysite.com
foo.bar.edu
best-site-ever.link
```

#### proxies.*.to

The protocol, domain, and port to proxy to. Should be an origin (scheme / hostname / port). Example values:

```
https://mysite.com/
http://localhost:8080/
http://127.0.0.1:123/
```

### redirects

A listing of domains to redirect.

```yaml
redirects:
  - from: my-old-site.com
    to: https://my-site.com
```

#### redirects.*.from

The domain to redirect from. Should be a domain name. Example values:

```
mysite.com
foo.bar.edu
best-site-ever.link
```

#### redirects.*.to

The base URL to redirect to. Should be an origin (scheme / hostname / port). Example values:

```
https://mysite.com/
http://localhost:8080/
http://127.0.0.1:123/
```

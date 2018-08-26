# Serving dPack over HTTP

## Usage

dWeb and dPack make it really easy to distribute live files on a HTTP server. This is a cool demo because we can also see how version history works! Serve dPack files on HTTP with the `--http` option. For example, `dweb sync --http`, serves your files to a HTTP website with live reloading and version history! This even works for dpacks you're downloading (add the `--thin` option to only download files you select via HTTP). The default HTTP port is 8080.

*Hint: Use `localhost:8080/?version=10` to view a specific version.*

Get started using dPack today with the `distribute` and `fork` commands.

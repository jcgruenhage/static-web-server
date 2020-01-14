# Static Web Server [![Build Status](https://travis-ci.com/joseluisq/static-web-server.svg?branch=master)](https://travis-ci.com/joseluisq/static-web-server) [![](https://images.microbadger.com/badges/image/joseluisq/static-web-server.svg)](https://microbadger.com/images/joseluisq/static-web-server "Get your own image badge on microbadger.com") [![Docker Image](https://img.shields.io/docker/pulls/joseluisq/static-web-server.svg)](https://hub.docker.com/r/joseluisq/static-web-server/)

> A blazing fast static files-serving web server powered by [Rust Iron](https://github.com/iron/iron). :zap:

**Static Web Server** is a very small and fast web server to serving static files such as html files or assets.

## Features

- Built with [Rust](https://rust-lang.org) which is focused on [safety, speed, and concurrency](https://kornel.ski/rust-c-speed).
- Memory safety and reduced overhead of CPU and RAM resources.
- Blazing fast static files-serving thanks to [Rust Iron](https://github.com/iron/iron).
- Suitable for small [GNU/Linux Docker containers](https://hub.docker.com/r/joseluisq/static-web-server). It's a fully __4,3M__ static binary thanks to [Rust and Musl libc](https://doc.rust-lang.org/edition-guide/rust-2018/platform-and-target-support/musl-support-for-fully-static-binaries.html) (`x86_64-unknown-linux-musl`).
- Gzip compression by default.
- Cache control headers included.
- Configurable via environment variables or CLI arguments.
- TLS support via [Rust Native TLS](https://docs.rs/native-tls/0.2.3/native_tls/) crate.
- Lightweight logging support.
- MacOs (`x86_64-apple-darwin`) binary support thanks to [Rust Linux / Darwin Builder](https://github.com/joseluisq/rust-linux-darwin-builder).
- [Scratch](https://hub.docker.com/_/scratch) and [latest Alpine Linux](https://hub.docker.com/_/alpine) Docker images available.

## Usage

Server can be configured either via environment variables or command-line arguments.

### Environment Variables

| Variable | Description | Default |
| --- | --- | --- |
| `SERVER_NAME` | Name for server. | Default `my-static-server`. |
| `SERVER_HOST` | Host address (E.g 127.0.0.1). | Default `[::]`. |
| `SERVER_PORT` | Host port. | Default `80`. |
| `SERVER_ROOT` | Root directory path of static files. | Default `./public`. |
| `SERVER_ASSETS` | Assets directory path for add cache headers functionality. | Default `./assets` but relative to the root. |
| `SERVER_ERROR_PAGE_404` | HTML file path for 404 errors. | If path is not specified or simply don't exists then server will use a generic HTML error message. Default `./public/404.html`.
| `SERVER_ERROR_PAGE_50X` | HTML file path for 50x errors. | If path is not specified or simply don't exists then server will use a generic HTML error message. Default `./public/50x.html` |
| `SERVER_TLS` | Enables TLS/SSL support. Make sure also to adjust current server port. | Default `false` |
| `SERVER_TLS_PKCS12` | A cryptographic identity [PKCS #12](https://docs.rs/native-tls/0.2.3/native_tls/struct.Identity.html#method.from_pkcs12) bundle file path containing a [X509 certificate](https://en.wikipedia.org/wiki/X.509) along with its corresponding private key and chain of certificates to a trusted root. | Default empty |
| `SERVER_TLS_PKCS12_PASSWD` | A specified password to decrypt the private key. | Default empty |

### Command-line arguments

CLI arguments listed with `static-web-server -h`.

```
static-web-server 1.4.0
A blazing fast static files-serving web server powered by Rust Iron

USAGE:
    static-web-server [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
        --assets <assets>
            Assets directory path for add cache headers functionality [env: SERVER_ASSETS=]  [default: ./assets]

        --host <host>                              Host address (E.g 127.0.0.1) [env: SERVER_HOST=]  [default: [::]]
        --name <name>                              Name for server [env: SERVER_NAME=]  [default: my-static-server]
        --page404 <page404>
            HTML file path for 404 errors. If path is not specified or simply don't exists then server will use a
            generic HTML error message [env: SERVER_ERROR_PAGE_404=]  [default: ./public/404.html]
        --page50x <page50x>
            HTML file path for 50x errors. If path is not specified or simply don't exists then server will use a
            generic HTML error message [env: SERVER_ERROR_PAGE_50X=]  [default: ./public/50x.html]
        --port <port>                              Host port [env: SERVER_PORT=]  [default: 80]
        --root <root>
            Root directory path of static files [env: SERVER_ROOT=]  [default: ./public]

        --tls <tls>                                Enables TLS/SSL support [env: SERVER_TLS=]
        --tls-pkcs12 <tls-pkcs12>
            A cryptographic identity PKCS #12 bundle file path containing a X509 certificate along with its
            corresponding private key and chain of certificates to a trusted root [env: SERVER_TLS_PKCS12=]  [default: ]
        --tls-pkcs12-passwd <tls-pkcs12-passwd>
            A specified password to decrypt the private key [env: SERVER_TLS_PKCS12_PASSWD=]  [default: ]
```

## TLS/SSL

TLS/SSL support is provided by [Rust Native TLS](https://docs.rs/native-tls/0.2.3/native_tls/struct.Identity.html#method.from_pkcs12) crate which supports [PKCS #12 cryptographic identity](https://en.wikipedia.org/wiki/PKCS_12).
An identity is an [X509 certificate](https://en.wikipedia.org/wiki/X.509) certificate along with its corresponding private key and chain of certificates to a trusted root.

For instance, identity files (`.p12` or `.pfx`) can be generated using the [OpenSSL SSL/TLS Toolkit](https://www.openssl.org/docs/manmaster/man1/pkcs12.html):

Generate a self-signed certificate:

```sh
openssl req -x509 -newkey rsa:4096 -nodes -keyout local.key -out local.crt -days 3650
```

Generate a PKCS #12 indentity file:

```sh
openssl pkcs12 -export -out identity.p12 -inkey local.key -in local.crt -password pass:my_password
```

## Docker stack

Example using [Traefik proxy](https://traefik.io/):

```yaml
version: "3.3"

services:
  web:
    image: joseluisq/static-web-server:latest
    environment:
        - SERVER_NAME=my-server
        - SERVER_HOST=127.0.0.1
        - SERVER_PORT=80
        - SERVER_ROOT=/html
        - SERVER_ASSETS=./assets
    volumes:
        - ./some-dir-path:/html
    labels:
        - "traefik.enable=true"
        - "traefik.frontend.entryPoints=https"
        - "traefik.backend=localhost_dev"
        - "traefik.frontend.rule=Host:localhost.dev"
        - "traefik.port=80"
    networks:
        - traefik_net

networks:
    traefik_net:
        external: true
```

## Contributions

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in current work by you, as defined in the Apache-2.0 license, shall be dual licensed as described below, without any additional terms or conditions.

Feel free to send some [Pull request](https://github.com/joseluisq/static-web-server/pulls) or [issue](https://github.com/joseluisq/static-web-server/issues).

## License

This work is primarily distributed under the terms of both the [MIT license](LICENSE-MIT) and the [Apache License (Version 2.0)](LICENSE-APACHE).

© 2019 [Jose Quintana](https://git.io/joseluisq)

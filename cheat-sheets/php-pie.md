# PIE cheatsheet

A highly opinionated [PIE](https://github.com/php/pie) cheat sheet.

PIE is the PHP extension manager, to get `redis`, `xdebug`, etc on your local machine.

While its usage might seem limited when thinking about extensions that can be obtained through
a distribution's package manager (APT, homebrew), PIE's main strength is that it allows anyone
to create an extension, make it available on packagist, and for all to install it with PIE.

## Install

First get the PIE "binary" (phar):

```console
# These dependencies are required by PIE, for it to be able to compile and install extensions
sudo apt-get install php autoconf automake libtool m4 make gcc
brew install         php autoconf automake libtool m4 make gcc

curl -fsSL -O https://github.com/php/pie/releases/latest/download/pie.phar \
        && sudo mv pie.phar /usr/local/bin/pie \
        && sudo chmod +x /usr/local/bin/pie
```

## Usage

* `pie install <vendor/project>:<version` to install a specific extension
  (similar syntax to composer)
* `pie uninstall <vendor/project` to uninstall
* `pie show` to list all installed extensions via PIE (`--all` to also show the ones already installed)

## Docker

It's possible to use PIE to install extensions in a docker image,
but that'll create a specific layer:

```
FROM php:8.5-cli-alpine AS php_dev_container

# autoconf bison gcc libtool make re2c: for PIE
RUN apk add --update --no-cache \
        autoconf bison gcc libtool make re2c \
        bash \
        curl-dev \
        libzip-dev \
        postgresql-dev

COPY --from=ghcr.io/php/pie:bin /pie /usr/bin/pie

# We use --force because these are "official" extension, so not really the focus of pie
RUN pie install --force php/curl \
    && pie install --force php/pdo_pgsql
```

It's probably better to use instead `docker-php-ext-install`:

```
FROM php:8.5-cli-alpine AS php_dev_container

RUN apk add --update --no-cache \
        bash \
        curl-dev \
        libzip-dev \
        postgresql-dev \
    && docker-php-ext-install \
        curl \
        pdo_pgsql
```

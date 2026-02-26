# PHP cheat sheet

A highly opinionated PHP cheat sheet.

* [config](#config)
* [array_merge](#array_merge)
* [latest version on Ubuntu](#latest-version-on-ubuntu)

## Config

Settings that usually need to be overwritten in `/etc/php/$PHP_VERSION/*/conf.d/99-custom.ini`:

```
;; ─────────────────────────────────────────────────────────────────────────────
;; 🔒🐘 Security.
;; ─────────────────────────────────────────────────────────────────────────────
[PHP]
expose_php = Off
display_errors = Off
allow_url_include = Off
session.cookie_httponly = 1
session.use_strict_mode = 1
session.cookie_secure = 1
session.cookie_samesite = Strict
session.use_trans_sid = 0
session.sid_length = 128
session.sid_bits_per_character = 6

;; ─────────────────────────────────────────────────────────────────────────────
;; 🕐🐘 Date configuration.
;; ─────────────────────────────────────────────────────────────────────────────
[Date]
date.timezone = UTC

;; ─────────────────────────────────────────────────────────────────────────────
;; ⬆️🐘 Upload size.
;; ─────────────────────────────────────────────────────────────────────────────
[Upload]
upload_max_filesize = 21M
post_max_size = 25M

;; ─────────────────────────────────────────────────────────────────────────────
;; 🐆🐘 OPcache.
;; PHP compiles every script into intermediate bytecode before executing it.
;; OPcache caches this bytecode in shared memory, skipping the compilation step
;; (which can take several hundreds of milliseconds) for all workers handling requests.
;; Introduced in PHP 5.5, enabled by default since PHP 8.5.
;;
;; ⚠️🐋 In the official Docker images for PHP <= 8.4, opcache needs to be installed explicitly:
;;
;; ```
;; FROM php:8.4.17-apache2-alpine
;; RUN docker-php-ext-install opcache
;; ```
;;
;; Evaluate if OPcache is configured efficiently:
;; * `opcache_get_status()`
;; * `phpinfo()`
;; ─────────────────────────────────────────────────────────────────────────────
[Opcache]
opcache.enable = 1

; Almost NEVER makes sense in the CLI
opcache.enable_cli = 0

; Size of the shared OPcache memory storage, in megabytes. Default: `128`, min: `8`
; There's only one OPcache per process.
; `opcache_get_status()['memory_usage']` if `used_memory` is very high and `free_memory` is very low (< 20%): increase
; `opcache_get_status()['opcache_statistics']` if `misses` is higher than `hits`: increase
; `opcache_get_status()['opcache_statistics']` if `opcache_hit_rate` isn't 99.999%: increase
opcache.memory_consumption = 256

; The part of `opcache.memory_consumption` dedicated to interned strings, in megabytes. Default: `8`, max: `opcache.memory_consumption`
; Stores hardcoded strings and symbol names (classes, functions, constants, methods)
; once in shared memory, reused across all parallel requests (without it, every request allocates them again)
; `opcache_get_status()['interned_strings_usage']` if `free_memory` close to 0: increase
opcache.interned_strings_buffer = 32

; Maximum number of keys (and scripts) in the hash table. Default: `10_000`, max: `1_000_000`
; With frameworks, the more the better, no harm setting it at 100_000 or more
; `phpinfo()` if `Cached keys` close to `Max keys`: increase
opcache.max_accelerated_files = 100000

; Checks if script modification time is more recent than cache one. Default: `1` (boolean)
; Set to `0` in an environment where the code never changes (e.g. not deploying over existing app)
opcache.validate_timestamps = 1
; How often `opcache.validate_timestamps` runs, in seconds. Default: `2`
; This is ignored ig `opcache.validate_timestamps` is set to `0`. 
opcache.revalidate_freq = 0 ; `0` means check on every request


; `opcache.preload` only makes sense if:
; * single application on server / container
; * AND using Nothing Shared Architecture (e.g. PHP-FPM, but not FrankenPHP worker mode)
; * AND you have a preload script (e.g. Symfony makes one for you)
```

Additional settings for PHP-FPM:

```
; For performance
pm = dynamic
pm.max_children = 8
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3
```

References:
* [how to properly and securely configure the entire php.ini](https://github.com/danehrlich1/very-secure-php-ini)
* [Everything about OPcache to increase PHP performance](https://www.youtube.com/watch?v=LcIkUpcaXZc)
* [OPcache Preloading - A miracle cure to improve PHP performance?](https://www.youtube.com/watch?v=wkhpNd7aYV8)
* [Properly restart Opcache after deployment, if not using containers](https://tideways.com/profiler/blog/properly-restart-opcache-after-deployment)

## array_merge

`+` operator will merge arrays, in case of duplicate keys, preserves the element from the **left-hand** array:

```
[1, 2, 'foo' => 'bar'] + [3, 4, 5, 'foo' => 'baz'] // [1, 2, 5, 'foo' => 'bar']
```

`array_merge` will merge arrays, in case of duplicate keys, overwrites the element using the **right-hand** array:

```
[1, 2, 'foo' => 'bar'] + [3, 4, 5, 'foo' => 'baz'] // [1, 2, 3, 4, 5, 'foo' => 'baz']
```

`array_merge` + "splat" operator will reduce a bidimensonal array to a flattened array:

```
array_merge(... [[1, 2], [3], [4, 5]]); // [1, 2, 3, 4, 5];
```

_References:_

* [StackOverflow: Difference between `+` and `array_merge`](http://stackoverflow.com/a/2140094/3437428)
* [PHP: Array operators](http://php.net/manual/en/language.operators.array.php)
* [PHP: `array_merge`](http://php.net/manual/en/function.array-merge.php)
* [PHP: `array_merge` splat operator comment](https://www.php.net/manual/en/function.array-merge.php#121023)

## Latest version on Ubuntu

* adding PPA: `sudo add-apt-repository ppa:ondrej/php -y; sudo apt-get update`
* removing current version: `sudo apt-get purge php* php*-fpm && sudo apt-get --purge autoremove`
* installing latest version (replace `X` with the latest version, eg `7.3`):
  `PHP_VERSION='X'; sudo apt-get install php$PHP_VERSION php$PHP_VERSION-curl php$PHP_VERSION-intl php$PHP_VERSION-pgsql php$PHP_VERSION-json php$PHP_VERSION-mbstring php$PHP_VERSION-zip php$PHP_VERSION-xml php$PHP_VERSION-fom apache2- apache2-bin-`


# Latest PHP version

* adding PPA: `sudo add-apt-repository ppa:ondrej/php -y; sudo apt-get update`
* removing current version: `sudo apt-get purge php* php*-fpm && sudo apt-get --purge autoremove`
* installing latest version (replace `X` with the latest version, eg `7.3`):
  `PHP_VERSION='X'; sudo apt-get install php$PHP_VERSION php$PHP_VERSION-curl php$PHP_VERSION-intl php$PHP_VERSION-pgsql php$PHP_VERSION-json php$PHP_VERSION-mbstring php$PHP_VERSION-zip php$PHP_VERSION-xml php$PHP_VERSION-fom apache2- apache2-bin-`
Settings that usually need to be overwritten in `/etc/php/$PHP_VERSION/*/conf.d/99-custom.ini`:

```
; For security reason
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

; For \date() and \DateTime()
date.timezone = UTC

; For file upload
upload_max_filesize = 21M
post_max_size = 25M

; For performance
opcache.enable=1
opcache.enable_cli=1
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

References: [how to properly and securely configure the entire php.ini](https://github.com/danehrlich1/very-secure-php-ini)

# Satis

[Composer](https://getcomposer.org/) relies on "satis" repositories that provides the required libraries.
[Packagist](https://packagist.org/) is a public one, but it's possible to host a private on for internal libraries.

## Installation instructions

Reference: [Satis: building your own Composer repository|http://blog.servergrove.com/2015/04/29/satis-building-composer-repository/]

### PHP

In order to run a Satis server, we need PHP >=5.3:

    sudo apt-get install php5

### Setting up Satis

To install it:

    curl -sS https://getcomposer.org/installer | php
    sudo mv composer.phar /usr/local/bin/composer
    composer create-project composer/satis:^1.0@alpha
    cd satis

Then we need to create a `satis.json` file:

```
{
    "name": "Acme",
    "homepage": "http://example.com/satis",
    "repositories": [],
    "require-all": true,
    "require-dependencies": true,
    "archive": {
        "directory": "dist",
        "format": "tar",
        "skip-dev": true
    }
}
```

Finally we need to set up the following cronjob:

```
# Update satis "Constant Commerce Repository" every 5 minutes
*/5 * * * * /usr/bin/env php /home/foobar/satis/bin/satis build --no-interaction /home/foobar/satis/satis.json /home/foobar/satis/acme-repository
```

### Apache

We need a web server, so we'll install Apache:

    sudo apt-get install apache2
    sudo a2enmod rewrite
    sudo a2enmod php5
    sudo a2enmod alias
    sudo a2enmod vhost_alias

Let's set up a virtual host, for example in `/etc/apache2/sites-available/example.com.conf`:

```
<VirtualHost *:80>
    ServerName example.com

    Alias /satis /home/foobar/satis/acme-repository
    <Directory /home/foobar/satis/acme-repository>
        DirectoryIndex index.html
        Options Indexes FollowSymLinks MultiViews
        AllowOverride all
        Require all granted
        Order allow,deny
        allow from all
    </Directory>
</VirtualHost>
```

If it's not the case already, we have to set Apache's user and group to ours:

```
# File: /etc/apache2/envvars
export APACHE_RUN_USER=foobar
export APACHE_RUN_GROUP=foobar
```

Finally we need to reboot Apache:

    sudo a2ensite example.com.conf
    sudo /etc/init.d/apache2 restart

## Adding a library

Adding a library to our Satis instance is done by adding an entry in the `repositories` attribute of the
`satis.json` file, for example:

```
{
    "name": "Acme",
    "homepage": "http://example.com/satis",
    "repositories": [
        {
            "type": "vcs",
            "url": "git@github.com:memio/spec-gen.git"
        }
    ],
    "require-all": true,
    "require-dependencies": true,
    "archive": {
        "directory": "dist",
        "format": "tar",
        "skip-dev": true
    }
}
```

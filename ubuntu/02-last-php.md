# Installing recent PHP on Ubuntu 13.04

[Ondřej Surý](https://launchpad.net/~ondrej) provides recent versions of PHP for Ubuntu.

Source: [Greg Rickaby - How to install LAMP on Ubuntu](http://gregrickaby.com/how-to-install-lamp-on-ubuntu/)

## PHP 5.5

    sudo add-apt-repository ppa:ondrej/php5
    sudo apt-get update
    sudo apt-get install php5

## PHP 5.4

    sudo add-apt-repository ppa:ondrej/php5-oldstable
    sudo apt-get update
    sudo apt-get install php5

# Installing last PHP 5.3 on Ubuntu 12.04

This article describes how to get the last PHP 5.3 version on Ubuntu 12.04,
using [Dotdeb instructions](http://www.dotdeb.org/instructions/).

## Use Dotdeb

Add these two lines to your `/etc/apt/sources.list` file:

    deb http://packages.dotdeb.org squeeze all
    deb-src http://packages.dotdeb.org squeeze all

Then fetch the appropriate GnuPG key:

    wget http://www.dotdeb.org/dotdeb.gpg
    cat dotdeb.gpg | sudo apt-key add -

Synchnonize your package index files:

    sudo apt-get update

## Installing the Internationalization extension

If you try to intall `php5-intl` you will probably encounter
the following error:

    php5-intl : Depends: libicu44 (>= 4.4.1-1) but it is not installable

To solve it, simply install the `libicu44.deb` directly from
[launchpad](https://launchpad.net/ubuntu/precise/+package/libicu44).

Then the extension can be installed:

    sudo apt-get install php5-intl

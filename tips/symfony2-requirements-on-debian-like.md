# Symfony2 requirements on Debian like

Version used:

* Ubuntu 13.04;
* Symfony 2.2.

## Minimum requirements

### PHP

    sudo apt-get install php5 php5-curl php5-intl

### Git as version control system

    sudo apt-get install git

### Composer as dependency manager

    curl -sS https://getcomposer.org/installer | php
    sudo mv composer.phar /usr/local/bin/composer

### Apache as HTTP server

    sudo apt-get install apache2
    sudo a2enmod rewrite

### ACL for permissions

First of all, install ACLs.

    sudo apt-get install acl
    
Then check which `sda` to edit. It should be the line ending with `Linux` (not `Linux swap / Solaris`):

    sudo fdisk -l | grep Linux

Now edit the file `/etc/fstab` to add `acl` to the options. Example:

    # <file system> <mount point>   <type>  <options>       <dump>  <pass>
    UUID=xxxxxxxxx /                ext4    [other options],acl 0       1

Finally, remount the `sda`:

    mount -a -o remount /dev/sdax

#### Changing Apache user and groups for permissions

If for some reasons you can't enable ACLs and you have Apache, change the user and group of the HTTP server.
By default, it should be `www-data`, change it to your user and group by editing the `/etc/apache/envvars` file.

In the following example, my user login is `gnugat` (so it will also be my group name):

    # Since there is no sane way to get the parsed apache2 config in scripts, some
    # settings are defined via environment variables and then used in apache2ctl,
    # /etc/init.d/apache2, /etc/logrotate.d/apache2, etc.
    export APACHE_RUN_USER=gnugat
    export APACHE_RUN_GROUP=gnugat

When restarting the Apache deamon (`/etc/init.d/apache2 restart`), it will run with your rights, which means it will
be able to read and right in your directories (example: `/home/gnugat/symfony2-project`).

### MySQL as database

    sudo apt-get install mysql-client mysql-server

Optionnaly:

    sudo apt-get install phpmyadmin

## CSS/Javascript minification

Download the last version of [node.js sources](http://nodejs.org/download/) and install it:

    tar zxvf node-v*.tar.gz
    cd node-v*
    ./configure
    make
    sudo make install
    cd ..
    rm -rf node-v*

### [UglifyJS](https://github.com/mishoo/UglifyJS) and [UglifyCSS](https://github.com/fmarcia/UglifyCSS)

    sudo npm install -g uglify-js@1
    sudo npm install -g uglifycss

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

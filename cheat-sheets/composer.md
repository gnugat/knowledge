# Composer cheatsheet

A highly opinionated [Composer](https://getcomposer.org/) cheat sheet.

## Version constraints

* `~2.6.1` translates to `>=2.6.1,<2.7.0`
* `~2.6` translates to `>=2.6.0,<2.7.0`
* `^2.6.1` translates to `>=2.6.1,<2.7.0`

## Commands

* download packages: `composer install -a`
* add package: `composer require <package>:<version>`
* update package: `composer update <package>`
* show installed version of package: `composer show <package> | grep version`

## Tips

* [Add a prefix for the autoloader configuration](https://github.com/gnugat/wizard-plugin/pull/4)
* [Use autoload-dev for tests](https://github.com/gnugat/redaktilo/pull/48#discussion_r16672357)

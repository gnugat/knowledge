# Composer cheatsheet

A highly opinionated [Composer](https://getcomposer.org/) cheat sheet.

## Version constraints

Tilde, allows patch updates:

* `~2.6.1`: locks the minor version (`>=2.6.1,<2.7.0`)
* `~2.6`: locks the major version (`>=2.6.0,<3.0.0`)

Caret, allows minor and patch updates:

* `^2.6.1`: locks the major version (`>=2.6.1,<3.0.0`)
* `^2.6`: locks the major version (`>=2.6.0,<3.0.0`)

## Commands

* `composer install`: download packages as defined in `composer.lock`
* `composer install --no-dev -a`: production install (no dev dependencies, optimised autoloader)
* `composer require [--dev] <package>:<version>`: add package (`--dev` for require-dev)
* `composer update -W <package>`: update package to latest version allowed by constraints (`-W` also updates its dependencies and their dependencies)
* `composer show <package> | grep version`: show installed version of package
* `composer dump-autoload --strict-psr --strict-ambiguous`: find files not complying with PSR autoloading (use `swiss-knife namespace-to-psr-4` to fix them)

Checks:

* `composer outdated -D`: list direct dependencies with newer versions available
* `composer audit`: check for known vulnerabilities

## Security

Since Composer 2.9, Composer refuses to install or update to a version
with a known vulnerability, unless you explicitly ignore the specific advisory.

Composer plugins execute code during installation.
They require explicit, name-based approval from the developer.

Dependencies cannot define post-install or pre-install scripts,
only the root project can.

## Recommended config

```json
{
    "config": {
        "optimize-autoloader": true,
        "bump-after-update": true,
        "sort-packages": true
    }
}
```

## Tips

* [Add a prefix for the autoloader configuration](https://github.com/gnugat/wizard-plugin/pull/4)
* [Use autoload-dev for tests](https://github.com/gnugat/redaktilo/pull/48#discussion_r16672357)

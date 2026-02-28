# Symfony versions cheat sheet

A highly opinionated Symfony versions cheat sheet:

* [Symfony 8.0](#symfony-80)

## Symfony 8.0

Requires PHP 8.4+. Released November 2025.
Symfony 8.0 = Symfony 7.4 - deprecations.

* `Console` [improved invokable commands (`#[Argument]`, `#[Option]`, `#[Ask]`, `#[MapInput]`, enum support)](https://symfony.com/blog/new-in-symfony-7-4-improved-invokable-commands)
* `Config` [new PHP array-based configuration format (`App::config()`)](https://symfony.com/blog/new-in-symfony-7-4-better-php-configuration)
* `Config` [XML configuration removed](https://symfony.com/blog/new-in-symfony-7-4-deprecated-xml-configuration)
* `ErrorHandler` [plain text exception rendering in terminal (instead of HTML)](https://symfony.com/blog/new-in-symfony-7-4-better-exceptions-in-terminal)
* `Uid` [UUID v7 by default, microsecond precision, `MockUuidFactory` for tests](https://symfony.com/blog/new-in-symfony-7-4-uid-improvements)
* `Runtime` [native FrankenPHP worker mode support (no extra package needed)](https://symfony.com/blog/new-in-symfony-7-4-dx-improvements-part-2#automatic-integration-of-frankenphp-worker-mode)
* `HttpFoundation` [Request: `get()` removed, body parsing for PUT/PATCH/DELETE, restricted method overrides](https://symfony.com/blog/new-in-symfony-7-4-request-class-improvements)
* `FrameworkBundle` [decoupled controller helpers (`ControllerHelper`, `#[AutowireMethodOf]`)](https://symfony.com/blog/new-in-symfony-7-4-decoupled-controller-helpers)
* `HttpClient` [caching HTTP client](https://symfony.com/blog/new-in-symfony-7-4-caching-http-client)
* `Security` [`#[IsGranted]` with `methods` option](https://symfony.com/blog/new-in-symfony-7-4-attribute-improvements#isgranted-methods)
* `Form` [multi-step forms (`AbstractFlowType`)](https://symfony.com/blog/new-in-symfony-7-4-multi-step-forms)
* `HttpKernel` [share directory (`kernel.share_dir`, for data shared across servers)](https://symfony.com/blog/new-in-symfony-7-4-share-directory)

Sources:
* [Symfony 8](https://symfony.com/8)
* [Symfony 8.0.0 released](https://symfony.com/blog/symfony-8-0-0-released)
* [Living on the edge - 8.0/7.4](https://symfony.com/blog/category/living-on-the-edge/8.0-7.4)

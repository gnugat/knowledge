# Building Native Apps with PHP

> **Talk Abstract**
>
> I'll show how we're building native mobile apps with PHP and go deeper into how it works

Speaker: Simon Hamp
* [Video](https://www.youtube.com/watch?v=ojy6LtaoRdI)

## Recap

Simon Hamp's talk is a live demonstration and pitch for NativePHP, a project he
has been building with his co-founder Shane.

The central claim is unusual enough to warrant a full explanation:

* this is not PHP that renders HTML inside a mobile shell
* and it is not PHP transpiled into something else.

It is PHP compiled natively for iOS and Android, running directly on the device.

### PHP as a glue language, taken to mobile

Simon's framing for why PHP is well-suited to this is worth repeating.

PHP's actual job in most systems is not to handle HTTP directly (that is the web
server's job) and not to store data (that is the database's job).
PHP is the glue between those systems. It wraps C, makes it more memory-safe and
approachable, and sits in the middle of everything.
That same role, Simon argues, transfers cleanly to mobile: PHP can sit between
the device's native APIs and the developer's application logic, doing what it has always done.

Nothing changes about how PHP behaves. NativePHP does not introduce a new build
target or a cross-platform abstraction layer. It compiles PHP separately for
iOS and for Android using each platform's own toolchain.

The resulting application runs natively on the device.

### Familiar tools, new platform

The developer experience is built on top of Laravel.
Routes, controllers, models, views, events, and Eloquent all work as expected.
Frontend code can use Vue, React, Svelte, or whatever the developer already reaches for.
There is a local SQLite database configured out of the box.
Applications run entirely offline by default, with no dependency on a network connection once installed.

Simon is direct about a practical detail: Xcode and Android Studio are technically
required to produce the native binaries, but NativePHP abstracts away the need
to actually use them. The developer writes PHP and runs a single command.

The getting-started flow he demonstrates live is:
1. `composer create-project` (or `laravel new`) to scaffold the project
2. One command to run the application via the Jump companion app
3. Scan a QR code on the device

The result is a running native application on a real phone, with no developer mode,
no provisioning profile, and no configuration.

### Edge: native UI without a web view

The talk distinguishes between two layers of UI. The web view layer is real:
NativePHP embeds a web view, and developers can use their standard HTML, CSS,
and JavaScript there.

But there is a second layer called Edge (Element Definition and Generation Engine)
that bypasses the web view entirely for certain components.

Edge uses Blade templates.
Developers declare components such as a top bar or a bottom navigation bar using
a syntax that resembles standard HTML. Edge compiles these templates into JSON
and passes that JSON to the native application layer.
The native layer uses the device's own rendering engine to produce components
that are indistinguishable from those in any other native app, because they are
those components.

The approach is similar to server-driven UI patterns used in large-scale native
applications, except that everything happens on-device.
There is no round trip to a server, no custom rendering code, and no compilation
step at build time. JSON generation in PHP is fast enough that the overhead is negligible.

### Plugin system

Any native API that NativePHP does not yet expose can be reached through its plugin system.
A plugin is a Composer package that contains:

- PHP code that calls the native layer
- Swift or Kotlin (or any other native language) code that implements the native side
- A `nativephp.json` manifest that wires the two together

When the application is built, the plugin's native code is compiled in alongside
the rest of the application. Installing a plugin is the same as installing any Composer package.

Simon announced a plugin marketplace at nativephp.com. Plugins are also published on Packagist.
He encouraged both open-source and commercial plugins, noting that building a
quality plugin is real work and premium pricing is appropriate for it.
NativePHP provides a plugin dev kit aimed at accelerating development,
with tooling built around Claude Code.

During the Q&A, plugins were confirmed as the answer for Apple HealthKit
integration and in-app purchases, with at least two contributors already working on the latter.

### Open source and roadmap

NativePHP for mobile launched at Laracon EU in February 2024 as a paid package.
A few weeks before this talk, Simon and Shane open-sourced it entirely.
The project is now free to use, maintained by the two of them full-time without large-organisation backing.

Items on the roadmap include:

- **Background tasks**: PHP running while the app is in the foreground doing other things,
    and eventually while the app itself is in the background
- **Full native UI**: eliminating the web view entirely and building UIs
    declaratively using Blade or a similar syntax, with reactive behaviour
    driven by the native rendering engine
- **Performance**: the current request-response cycle runs at around 100 milliseconds,
    which Apple considers acceptable for UI interactions.
    Proof-of-concept work has brought this down to 10 milliseconds

On the local-first question raised in Q&A: apps are already offline-capable by default.
What is in active development is the synchronisation layer for server-client setups,
intended to be turnkey: deploy an API server, get sync working locally with minimal configuration.

Apps built with NativePHP are already in the iOS App Store. Apple does allow it.

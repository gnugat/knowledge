# PHP without PHP: Make Standalone Binaries from Your Code

> **Talk Abstract**
>
> What if your PHP script became executable, with no dependance on the PHP server,
> or Composer?
>
> We can compile a PHP tool to a static binary, which can be executed anywhere,
> just like a Go or Rust tool.
>
> In this talk we'll see:
>
> - how to build PHP scripts ann libs into static binaries
> - what are the limits (extensions, size, compatibility...)
> - when PHP can replace Bash, Python, Go (concrete examples)
>
> Let's discover a PHP... that installs nowhere.

_Originally in French_
Speaker: Jean-François Lépine
* [Video](https://www.youtube.com/watch?v=gqre9g4XV00)

## Recap

Jean-François Lépine opens with two stories from his career.
About ten years ago he worked in French defense ministry buildings, auditing
systems on air-gapped networks.
Deploying PHP code meant handing a USB key to a cleared individual, walking
through an airlock to a separate network, following an 80-page deployment
document, watching it fail, and starting over.
Last year he encountered the same constraints while building a software
application for recycling center access barriers: no internet, mandatory
on-site visits to a factory in Belgium, a heavily restricted environment.
In the decade between the two projects, PHP had evolved enough to make a
different approach possible.

### Why standalone binaries

The problem with shipping PHP software is the dependency chain.
The target machine might have no network access, or network access through
restrictive proxies.
It might have internet but no rights to install software.
It might have PHP installed but at the wrong version, or with the wrong
extensions, or with the wrong system libraries.

A standalone binary eliminates all of that.
Drop one file, run it, done.
This is how Go and Rust tools are distributed, and it turns out PHP can do the
same.
The benefits extend to cloud deployments too: spin up a bare OS, copy the
binary, PHP runs without pulling gigabytes of Docker layers.

### A brief history of compiled PHP

Attempts at turning PHP into compiled artifacts go back to the early 2010s.
HipHop (a few hands in the audience recognized it) compiled PHP to C++ or .NET.
Those approaches were transpilers: they converted PHP source into another language.
What is different today is that the PHP interpreter itself is embedded in the binary.
It is real PHP, not a reimplementation.

### The embed SAPI

To understand how this works it helps to understand what PHP actually is.
PHP is a set of C functions, and the outside world needs a bridge to reach them.

That bridge is the SAPI (Server Application Programming Interface).

CGI, FPM, and the CLI are all SAPIs.
**The one that matters here is the embed SAPI**.

The embed SAPI allows any language that can call C to use PHP directly.
Go, Rust, or plain C can manipulate PHP source, call PHP functions, and get
results back.
This is the mechanism that makes embedding PHP into a native binary possible.

### Static PHP CLI and PHP Micro SFX

Getting the embed SAPI working used to require downloading the right PHP source,
the right versions of every dependency, configuring the build correctly, and
spending hours debugging compilation errors.

Two tools changed this.

**Static PHP CLI (SPC)**, created in 2020, abstracts all of that complexity.
It comes with a `doctor` command that checks whether the build environment has
everything it needs, and an `--auto-fix` flag that installs missing dependencies
automatically.
From there, one command downloads PHP 8.4 source and any required extensions.
Another builds a statically linked PHP binary with everything included.

**PHP Micro SFX** is the glue.
It takes a compiled PHP binary and a `.phar` archive and concatenates them
into a single executable.
The result is a file that carries its own PHP interpreter alongside the
application code.

These two tools are the foundation of most of what is happening with PHP
outside traditional web servers:
* FrankenPHP in static mode uses SPC
* Laravel Herd uses SPC
* NativePHP for mobile uses SPC

The recycling center access barrier (Jean-François's original use case) runs PHP with FrankenPHP.

### The build process

The full process involves a few steps.

First, package the application as a `.phar`.
This is straightforward with Box, which takes a project directory and produces
a `.phar` archive in two lines.

Then, on a machine with SPC set up:

```bash
# download PHP source and selected extensions
spc download php-8.4 --with-extensions=...

# build the micro SFX glue
spc build micro

# combine micro SFX and the .phar into a binary
spc micro:combine app.phar --output=app
```

The resulting binary targets the architecture it was built on.
Cross-compilation for Windows, macOS, ARM, or x86 requires separate builds.

Jean-François also mentions PHP-Pack, a newer tool he discovered while
preparing the slides.
It wraps the entire process into two lines and can build for all target
architectures in a single command.
He has limited experience with it but tested it successfully.

Optional: running the binary through UPX reduces its size further, though this
can cause issues in some cases and is worth testing before using in production.

### Constraints

The binary is real PHP.
The same source code, the same behavior, no emulation.
There are constraints worth knowing.

**FFI is not available.**
Foreign Function Interface calls, which allow PHP to invoke C libraries
directly, do not work in static builds.
For most CLI tooling this is not a problem.

**Binary size.**
A minimal PHP binary with no extensions is around 3 MB.
PHP Matrex with all the extensions Jean-François needed came out at 40 MB.
That is larger than a Go binary but still smaller than asking users to install
PHP, libxml, libu, and everything else.

**Startup time.**
PHP Matrex normally boots in around 40 milliseconds.
The static binary version takes around 80 milliseconds.
Two reasons: phar extraction adds overhead at startup, and the static build
uses musl libc rather than glibc, which is slower for some operations.
For a CLI tool where the user is waiting tenths of seconds, this is unlikely
to matter.
For something where 10 milliseconds is significant, it might not be the right
approach.

### Use cases

Jean-François considers this particularly well suited to tooling.
Castor, the Symfony console, Artisan: all of these could be distributed as
prebuilt binaries rather than requiring users to have a working PHP environment.

For web applications, the recommendation is to use FrankenPHP rather than
building a custom static PHP setup from scratch.
FrankenPHP handles the complexity and is already production-proven.

### A call to contribute

Static PHP CLI has around thirty contributors across five years of existence.
Jean-François considers it the foundation on which the future of PHP outside
the browser is being built, and invites the community to engage with it.
Testing it, adding support for an extension, or just starring the repository
all help keep the project alive.

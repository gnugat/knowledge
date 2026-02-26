# SymfonyOnline January 2026

* [Schedule](https://live.symfony.com/2026-online-january/schedule)

## Overview

SymfonyOnline January 2026 was a 16-talk conference
covering the state of Symfony as it approaches its 20th anniversary
and prepares for the Symfony 8 release.

Three major themes emerged across the talks:

**Performance and observability** dominated the programme.
Antoine Bluchet presented the new JSON Streamer, Object Mapper,
and compile-time code generation that push Symfony's serialization performance
to levels competitive with hand-written code.
Paul Dragoonis delivered a dense workshop-style talk on PHP runtime tuning:
OPcache, FPM, FrankenPHP worker mode, Redis with igbinary,
and Grafana dashboards to make it all observable.
Mitchel Vroege showed how to call Rust from PHP via FFI
to accelerate specific hot paths without rewriting the application.
Benjamin Eberlei explained how PHP 8.4's native lazy objects
will eliminate proxy code generation in Doctrine and Symfony,
with measurable startup time savings.

**Architecture and design** was the second thread.
Oskar Barcz made a case for pragmatic CQRS in Symfony:
separate command and query buses, but without event sourcing unless you actually need it.
Rob Allen's Strategic DDD talk focused on Bounded Contexts, Ubiquitous Language,
and Context Mapping as collaboration tools rather than code patterns.
Kris Wallsmith shared how Vacatia uses Symfony Messenger at scale:
multiple buses, message chaining, batching, and tracing across asynchronous workflows.

**Developer experience and tooling** tied the conference together.
Robin Chalas showcased the Console component's new attribute-based syntax,
making commands as clean to write as controllers.
Sebastian Bergmann clarified when to use test doubles and when to test with real dependencies.
Christopher Hertel demonstrated the Symfony AI components in action:
LLM integration, embeddings, vector stores, and model inference via PHP extensions.
Pauline Vos live-coded a WebSocket raffler with Ratchet,
proving that real-time PHP does not require complex infrastructure.
Nils Adermann laid out the security roadmap for Composer and Packagist:
artifact integrity, trusted publishing, and Sigstore attestations.
Anna Filina gave practical advice on surviving Symfony upgrades
by spreading deprecation fixes continuously rather than batching them.
Valentin Rusev dissected the first Git commit
to illustrate Gall's Law: every complex system that works evolved from a simple one.

## My highlights

The standout talk was Paul Dragoonis's
[From Runtime to Resilience: Scaling PHP](./024-from-runtime-to-resilience-scaling-php.md).
It was extremely dense, covering `php.ini` hardening, Composer autoloading,
Redis with igbinary, FPM tuning, OPcache internals, FrankenPHP worker mode,
and CQRS projections, all in a single session.
The workshop format made it feel less like a conference talk
and more like a guided deep-dive you'd want to replay at half speed
while replicating the steps locally.
The best value talk of the conference, hands down.

Robin Chalas's [Rediscover the Console](./011-rediscover-the-console.md)
was the other highlight.
The new attribute-based syntax for console commands
finally brings them on par with controllers in terms of developer experience.

Benjamin Eberlei's
[How native lazy objects will change Doctrine and Symfony forever](./021-how-native-lazy-objects-will-change-doctrine-and-symfony-forever.md)
was another standout.
Not the most practical talk for day-to-day work
(native lazy objects are mostly a framework concern),
but fascinating to learn how they work under the hood.
The Magento prototype showing 25-30% request time savings
makes the implications for Symfony's container hard to ignore.

Antoine Bluchet's
[2025: Performance Milestone for the Symfony Ecosystem](./002-2025-performance-milestone-for-the-symfony-ecosystem.md)
was very impressive.
JSON Streamer, Object Mapper, and compile-time code generation
push Symfony's serialization performance to levels competitive with hand-written code.

It was also a reminder that the long-time contributors are still at the top of their game.
Sebastian Bergmann's [Testing with(out) dependencies](./008-testing-with-without-dependencies.md)
was a masterclass in knowing when to use test doubles and when not to.
Fabien Potencier's [opening keynote](./001-20-years-of-symfony-whats-next.md)
and Nicolas Grekas's [closing talk](./030-a-year-of-symfony-hello-symfony-8.md)
bookended the conference with the kind of clarity and vision
you'd expect from twenty years of leading the project.

The biggest surprise was Christopher Hertel's
[Symfony AI in Action](./016-symfony-ai-in-action.md).
No fluff, no hype: it focused on the day-to-day need
of integrating AI features into everyday projects.
The Symfony AI components are further along than expected:
LLM integration, embeddings, vector stores,
and even model inference natively in PHP via the ORT extension.

## Overall impression

The conference had a clear Symfony 8 hype-train energy.
It opened with Fabien Potencier reflecting on 20 years of Symfony
and closed with Nicolas Grekas walking through a year of contributions
leading to the next major release.
All 16 talks were solid; none I'd skip on a rewatch.

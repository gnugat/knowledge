# A Year of Symfony, Hello Symfony 8

> **Talk Abstract**
>
> So many things happened in Symfony during the last 6 months!
> Let's look at what's new: three brand-new components (JsonStreamer, JsonPath, and ObjectMapper),
> expanding use of PHP attributes in Commands and Dependency Injection,
> and the ongoing work to consolidate configuration formats down to a single, tooling-friendly approach.
> And of course: hello Symfony 8!

* Similar to `20 years of Symfony, PHPverse 2025`, see [video](https://www.youtube.com/watch?v=7IVDUnvvhmc)

Speaker: Nicolas Grekas
* [Twitter/X profile](https://twitter.com/nicolasgrekas)

![Title slide](./a-year-of-symfony-hello-symfony-8/00-title.png)

## Recap

Nicolas Grekas' talk offers a brisk tour of the last six months of Symfony development, spanning the final feature release of the 7.x line and the first major in two years: Symfony 8.
The key framing is that Symfony 8 is intentionally "boring": all the new features landed in 7.4, and 8.0 is about removing deprecated code, requiring PHP 8.4, and benefiting from its quality-of-life improvements such as native lazy objects and improved type safety.
One notable exception to the "no new features" rule: versions ending in `.4` don't get new components, so 8.0 is where the new components land.

![Changes in the last 6 months slide](./a-year-of-symfony-hello-symfony-8/01-changes.png)

The numbers illustrate the pace: in just six months, Symfony 8 accumulated 4k files changed, +188/-78 features, 475 minor improvements, 513 merged PRs, and 342 bug fixes (and that's excluding the addition of the emoji database).

### JsonStreamer

The `JsonStreamer` component addresses a performance gap in the existing `Serializer`: for large JSON payloads, the standard encode/decode approach holds the entire document in memory.
JsonStreamer instead generates PHP code that reads or writes JSON in a streaming fashion, almost byte by byte, using PHP generators internally.

The public API is built around three interfaces (`StreamWriterInterface`, `StreamReaderInterface`, and `ValueTransformerInterface`), but for most use cases the entry point is simply annotating a DTO with `#[JsonStreamable]`.

![JsonStreamer slide](./a-year-of-symfony-hello-symfony-8/02-json-streamer.png)

The performance gains are significant: serialization is around 10x faster than the Serializer component with 50% less memory, and deserialization is around 10x faster with 90% less memory.
In practice, API Platform reported that the share of request time spent on serialization dropped from 83% to 45% after adopting it.

Two additional attributes round out the feature set.
`#[StreamedName]` customises the property name in the JSON output (equivalent to `#[SerializedName]` in the Serializer).
`#[ValueTransformer]` applies a transformation callable during encoding or decoding, accepting either a plain PHP function name or a static method reference.

Types are declared explicitly using the `TypeInfo` component (e.g. `Type::list(Type::object(User::class))`), which is what allows the code generator to produce specialised, allocation-free PHP for each type.

It is less flexible than the Serializer (no `#[Groups]` support, no recursive transformer application) but significantly more capable for performance-critical scenarios such as large API responses or bulk data imports.
The component shipped as experimental in Symfony 7.3.

### JsonPath

A companion `JsonPath` component shipped alongside JsonStreamer, implementing RFC 9535 for querying JSON documents.
It follows the same crawler pattern as the existing `DomCrawler` component for HTML/XML.

```php
$crawler = new JsonCrawler($json);

// basic path navigation
$crawler->find('$.store.book[0].title');

// descendant queries
$crawler->find('$..author');

// filter expressions
$crawler->find('$.store.book[?(@.price < 10)]');
$crawler->find('$.store.book[?match(@.author, "[A-Z].*el.+")]');
```

For programmatic query construction, a fluent `JsonPath` builder is also available, chainable via `->key()` and `->index()` calls.
All results are returned as arrays.

### ObjectMapper

The `ObjectMapper` component handles the common task of transferring data between objects of different classes, typically DTOs crossing the boundary between layers of an application.
Rather than writing manual mapping code property by property, the `#[Map]` attribute on the source class describes how each property should be transferred.

![ObjectMapper slide](./a-year-of-symfony-hello-symfony-8/03-object-mapper.png)

The class-level `#[Map]` attribute declares the target class.
At the property level, `target` renames the destination property, `transform` applies a callable before writing (e.g. `strtolower`), and `if` makes mapping conditional at runtime.
Mapping is then a one-liner: `$mapper->map($dto, User::class)` to create a new object, or `$mapper->map($dto, $existingUser)` to update one in place.

A `TargetClass` condition allows the same source class to map differently depending on the destination, for example omitting sensitive fields when mapping to a public profile class.
The `ObjectMapperInterface` is automatically autowired.

The component also handles recursive and cyclic object graphs, and supports services for complex transformation or condition logic.
It shipped as experimental in Symfony 7.3.

### Commands as DTOs

The trend of reducing boilerplate through PHP attributes continues in the Console component.
Commands no longer need to extend the `Command` class: a plain class with `#[AsCommand]` and an `__invoke()` method is enough.
In the simple form, `#[Argument]` and `#[Option]` go directly on the `__invoke` parameters, with types and default values determining whether each is required or optional.

For richer interaction, the talk highlights the DTO approach: `__invoke` accepts a single DTO annotated with `#[MapInput]` (which tells Symfony to populate it from the command's input), and all the argument/option/prompt declarations live on the DTO's properties.

![Commands using attributes slide](./a-year-of-symfony-hello-symfony-8/04-command-attributes.png)

`#[Argument]` maps a property to a positional argument, `#[Ask]` attaches a prompt label (with an optional `hidden: true` for passwords), and `#[Interact]` marks a public method to run during the interaction phase (replacing the previous `extend Command` + override `interact()` pattern).
The example shown generates a random password and copies it to the clipboard when the user skips that field.

![Commands DTO slide](./a-year-of-symfony-hello-symfony-8/05-commands-dto.png)

`#[Argument]` and `#[Option]` also accept backed enums: the user's input string is automatically converted to the corresponding enum case, and an invalid value produces a clear error listing all supported values.

`CommandTester` is fully compatible with all these new patterns, so commands remain straightforward to test.
The classic `Command` extension approach is not deprecated; both styles coexist.

### DependencyInjection: #[AutowireMethodOf]

A new `#[AutowireMethodOf]` attribute allows a controller (or any service) to inject individual methods of a helper class as `\Closure` instances, rather than injecting the whole helper class.

![Controller inject method of slide](./a-year-of-symfony-hello-symfony-8/06-controller-inject-method-of.png)

This is an alternative to extending `AbstractController`: instead of inheriting `render()` and `redirectToRoute()` from a parent, a controller explicitly declares which helper methods it needs.
The result is less coupling to the controller infrastructure and more expressive dependency declarations.

### Auto-discovering Value Objects

The Dependency Injection container has always scanned the source tree to discover services, but it ignored value objects since they are not meant to be wired as services.
A new `Definition::addResourceTag()` method makes it possible to register a scan for those classes and attach tags to them.

![Auto-discover value objects slide](./a-year-of-symfony-hello-symfony-8/07-autodiscover-value-objects.png)

The immediate benefit is that bundles like Doctrine ORM can use this mechanism to automatically discover Entity classes without any additional configuration.
As Ryan Weaver commented on the PR: _"I can't wait to see what we do with `#[AppModel]`!! This one seems subtle but we're gonna feel its impact!"_

### Configuration: down to one format

Symfony has historically supported XML, YAML, and PHP as configuration formats.
The talk signals a deliberate consolidation: XML was deprecated in Symfony 7.4 and removed entirely in 8.0 (with the `ext-xml` requirement potentially following).
YAML remains the default and is used by all Symfony recipes, so it cannot be removed yet, but the direction is clear.

![Drop XML config slide](./a-year-of-symfony-hello-symfony-8/08-drop-xml-config.png)

The push is toward PHP as the preferred format, for concrete tooling reasons: autocompletion, static analysis, `use` statements, automated patching, and backward-compatibility friendliness.
The fluent config builder classes introduced in Symfony 5.3 have been deprecated: they cannot represent all valid configuration shapes, and they are incompatible with Symfony Flex's automated recipe updates.
Their replacement is YAML-like nested arrays passed to `App::config()`.

![PHP slide](./a-year-of-symfony-hello-symfony-8/09-php-config.png)

![YAML-like arrays for config files slide](./a-year-of-symfony-hello-symfony-8/10-yaml-like-objects.png)

The canonical shape for every bundle's configuration is documented in a generated `config/reference.php` file, which contains the array shapes for all installed bundles and enables IDE autocompletion and static analysis.
It is recommended to commit this file; it can also be added to the `classmap` in `composer.json` to make the shapes available to static analysers.
Environment-specific configuration uses a `when@{env}` key inside the array, keeping everything in one file.

![Config reference slide](./a-year-of-symfony-hello-symfony-8/11-config-reference.png)

YAML is not left behind either: JSON schemas are now available for services, routes, validation, and serialization configuration files, enabling real-time validation and autocompletion in IDEs that support the `yaml-language-server` schema hint.
An automated [XML-to-PHP conversion tool](https://github.com/GromNaN/symfony-config-xml-to-php/) is available for bundle authors migrating away from XML.

This work is still in progress: YAML remains the default and Flex still only understands YAML as of Symfony 8.0, so the new PHP format is not yet the recommended choice for most projects.
But the trajectory is clear: one format, full tooling support, no more config format debates.

### Thank you to the community

Nicolas closes with a reminder that none of this happens in a vacuum.
All of the features covered in the talk are the result of contributions from the broader Symfony community: pull requests, reviews, bug reports, documentation, and the countless conversations at conferences and on Slack that turn rough ideas into shipped features.

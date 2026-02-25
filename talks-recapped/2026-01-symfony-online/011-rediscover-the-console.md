# Rediscover the Console

> **Talk Abstract**
>
> Console is one of the most widely used Symfony components and ranks among the top 5 most downloaded PHP packages worldwide.
>
> Used by essential tools such as Composer, PHPStan, and Psalm, as well as by most PHP frameworks, CMS, e-commerce platforms, and ultimately our end applications, the console is ubiquitous.
>
> The more a package is used, the more difficult it is to evolve. The slightest bug fix on the Console can disrupt millions of users. Nevertheless, the component has been constantly improved thanks to the countless contributions it has received since its introduction in 2010, while remaining true to its promise of backward compatibility.
>
> But it's time to give the Console a major facelift, not only to get rid of some limitations and other design issues, but also to open it up to new possibilities.
> That's why a few key contributors have been working hard to redesign it.
>
> It's available since Symfony 7.3, and will get even better in a future version.
> Join me to rediscover the Console!

Speaker: Robin Chalas
* [Slides](https://speakerdeck.com/chalasr/rediscover-the-console-symfonycon-amsterdam-2025)

## Recap

Robin Chalas opens with a striking parallel: **Console commands are to the Console what controllers are to HTTP**. They are the same concept, and so, one would assume, they should be equally powerful. Until very recently, they were not. It is impressive to see how drastically one of Symfony's oldest components has evolved over just the last year.

### The Console, by the numbers

The Symfony Console component is the 6th most used PHP package worldwide in production, according to a recent PHP Foundation ecosystem analysis. That translates to one billion cumulative installs and 500,000 new installs per day.

What makes those numbers so high is not just end applications depending on the Console, but all the critical tooling those applications already rely on: Laravel, API Platform, PHPStan, Psalm, and most notably Composer itself. When you run `composer require`, you are running a Symfony command.

That ubiquity is precisely what made evolving the component so difficult. Every bug fix risks disrupting millions of users and triggering complaints. Composer cannot break. The pressure of backward compatibility kept the component frozen in place for years.

### The legacy: extend, configure, execute

Before Symfony 7.3, writing a command meant following a strict inheritance-based contract: a class extending `Command`, a `configure()` method to declare input arguments and options, and an `execute()` method containing the business logic.

```php
#[AsCommand(name: 'app:create-user')]
class CreateUserCommand extends Command
{
    protected function configure(): void
    {
        $this->addArgument('name', InputArgument::REQUIRED);
        $this->addOption('activate', null, InputOption::VALUE_NONE);
    }

    public function execute(InputInterface $input, OutputInterface $output): int
    {
        $io = new SymfonyStyle($input, $output);
        $name = $input->getArgument('name');
        $activate = (bool) $input->getOption('activate');
        // ...
        return Command::SUCCESS;
    }
}
```

This is our legacy. Several problems compound.

First, composition over inheritance: extending a base class with a large, messy public API (`addArgument()`, `addOption()`, `setName()`, and many more) is exactly what object-oriented design principles argue against. Controllers abandoned forced inheritance years ago. Commands had not.

Second, the boilerplate is tedious and repetitive. Defining an argument in `configure()` and then retrieving it in `execute()` means writing the same name twice, with no shared type information between the two calls. The input API returns untyped values, forcing developers to add PHPDoc annotations just to teach static analysers what type they will actually receive. Controllers handled this problem long ago with `MapRequestPayload` and `MapQueryParameter`. The Console was left behind.

### The new way: invokable commands (Symfony 7.3)

Symfony 7.3 changes this entirely. A command no longer needs to extend anything. It can be any PHP class, as long as it implements `__invoke()`.

```php
#[AsCommand(name: 'app:create-user')]
class CreateUserCommand
{
    public function __invoke(
        SymfonyStyle $io,
        #[Argument] string $name,
        #[Option] bool $activate = false,
    ): int {
        // ...
        return Command::SUCCESS;
    }
}
```

The `#[AsCommand]` attribute still defines the name and description. The constructor still declares dependencies through normal injection. But `configure()` disappears entirely.

Arguments and options are declared as parameters of `__invoke()`, each annotated with `#[Argument]` or `#[Option]`. The method signature is now both the declaration and the retrieval point: typed, readable, and concise.

Dependency injection applies to the invocation itself too. A `SymfonyStyle` instance, an `InputInterface`, or an `OutputInterface` can all be requested directly as `__invoke()` parameters. Symfony resolves them automatically based on the method signature.

Both attributes accept optional configuration:

```php
public function __invoke(
    // rename the argument or option
    #[Argument(name: 'identifier', description: 'The user login or email address')] string $name,
    // optional argument with a default value
    #[Argument] int $retries = 3,
    // option with a shortcut and suggested values
    #[Option(shortcut: 'v', suggestedValues: [self::class, 'getSuggestedRoles'])] array $roles = ['ROLE_USER'],
): int { /* ... */ }
```

Robin summed up the result with the best line of the talk: **"The difference is that it fits on a slide now."** And it is much more readable.

The old class-based style still works and is not deprecated. Both approaches can coexist in the same application.

### How it works internally

The implementation does not introduce a new component. It builds on `Command::setCode()`, a method that has existed for years and accepts a callable in place of an `execute()` implementation.

Internally, `setCode()` now wraps any callable in an `InvokableCommand`, an internal class that handles argument resolution: reading the method signature, identifying which parameters correspond to Console arguments, which to options, and which to framework-provided objects like `SymfonyStyle`, then mapping raw input values to the typed parameters accordingly.

The one limitation in Symfony 7.3 was the absence of `interact()` support, the lifecycle method that allows commands to prompt the user interactively before execution. Robin polled the audience; two people raised their hand as having `interact()` in production. The trade-off seemed acceptable.

### Migrating your commands

Rector, the automated refactoring tool, already has a rule to migrate legacy commands to the new invokable style. Robin encourages everyone to use it. The new API is where the Symfony core team's focus will be in the coming years.

### Symfony 7.4: fewer limitations, more capabilities

Several improvements landed in Symfony 7.4.

**BackedEnum inputs**: arguments and options can now be typed as `BackedEnum`. The Console resolves the raw string input to the matching enum case automatically, and invalid values produce a clear error message listing the supported options.

![Enums support: two enum definitions on the left, an invokable command using them as typed #[Argument] and #[Option] on the right](./011-rediscover-the-console/00-enum-support.png)

```php
enum CloudRegion: string { case East = 'east'; case West = 'west'; }
enum ServerSize: int { case S = 1; case M = 2; case L = 3; }

#[AsCommand('app:add-server')]
class AddServerCommand
{
    public function __invoke(
        OutputInterface $output,
        #[Argument] CloudRegion $region,
        #[Option] ?ServerSize $size = null,
    ): int { /* ... */ }
}
```

**`#[MapInput]` for DTOs**: a new `#[MapInput]` attribute allows defining a separate PHP class whose public properties carry the `#[Argument]` and `#[Option]` attributes. The Console maps the raw input onto that class and injects the populated object into `__invoke()`. The same pattern as `MapRequestPayload` in controllers, applied to CLI input.

```php
class AddServerInput
{
    #[Argument] public CloudRegion $region;
    #[Option] public ?ServerSize $size = null;
}

#[AsCommand('app:add-server')]
class AddServerCommand
{
    public function __invoke(
        OutputInterface $output,
        #[MapInput] AddServerInput $server,
    ): int { /* ... */ }
}
```

**`interact()` support restored**: a `#[Interact]` attribute marks a public method on the command class as the interactive input phase, and a `#[Ask]` attribute can be placed directly on a parameter or DTO property to prompt the user automatically when that value is missing. This removes the last excuse for not migrating.

```php
#[AsCommand('app:add-server')]
class AddServerCommand
{
    #[Interact]
    public function prompt(InputInterface $input, SymfonyStyle $io): void
    {
        if (!$input->getArgument('region')) {
            $input->setArgument('region', $io->ask('Enter the cloud region name'));
        }
    }

    public function __invoke(
        #[Argument, Ask('Enter the cloud region name')] CloudRegion $region,
    ): int { /* ... */ }
}
```

**Standalone usage**: invokable commands previously required the full Symfony framework with its DI container. A new `Application::addCommand()` method makes them available to tools that use only the Console component in isolation, like PHPStan, Psalm, and Composer, without requiring the framework kernel.

### What comes next

Commands still do not have full feature parity with controllers. Two things are notably missing.

**Validator integration**: the ability to place Symfony Constraint attributes directly on `__invoke()` parameters, triggering validation before the command body runs.

**Command argument resolvers**: controllers have had `ArgumentValueResolver` extension points for years, enabling automatic Doctrine entity hydration from route parameters and `MapRequestPayload` deserialization. Robin has an open pull request exploring how to make this resolution system available to the Console as well. The difficulty is defining a clean abstraction that works equally well for HTTP requests and Console input. The current thinking is to build a parallel resolver system inside the Console component itself, the direction already taken with `MapInput`.

A small but concrete contribution opportunity: updating **maker-bundle** to scaffold invokable commands instead of the legacy class-based style. Robin called it out explicitly as a great first contribution.

Robin also half-jokingly raised the idea of a TUI (terminal user interface) component for Symfony. A proper name was suggested from the audience: **CroissanTUI**.

### The origin

Robin closes with the history. This work traces back to a 2019 European Commission hackathon in Brussels, where Robin met Yonel Ceruto for the first time. The idea of attribute-based command arguments was already circulating then. Robin even presented it at a Paris event three years ago hoping to implement it himself. Yonel got there first, faster and better. "I built a company in the meantime," Robin noted.

The result is a Console component that, fifteen years after its introduction, finally treats commands as first-class objects on equal footing with controllers: simpler to write, easier to read, and ready for the next decade.
A huge thank you to everyone who contributed to the Console DX, and especially to Yonel Ceruto.

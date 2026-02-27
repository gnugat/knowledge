# PHPUnit cheat sheet

A highly opinionated [PHPUnit](https://phpunit.de/) cheat sheet.

## Assertions

Prototype: `$this->assertX($expected, $actual, $message = '')`

* `assertSame`: strict equality (`===`)
* `assertEquals`: loose equality (`==`)
* `assertTrue`, `assertFalse`
* `assertInstanceOf`

Exceptions:

* `expectException(\Exception::class)`: should throw exception `\Exception`
* `expectNotToPerformAssertions()`: should not throw exception

## Attributes

Attributes (`#[<Name>]`) replace Annotations (PHPDoc `@<Name>`),
deprecated in PHPUnit 10, removed in PHPUnit 12.

Specify targeted class:

* `#[CoversClass(MyClass::class)]`: enforces test boundaries, prevents accidental coverage
* `#[UsesClass(OtherClass::class)]`: if code from other classes is expected to be used

Categorize tests:

* `#[Small]`: unit tests (under 100ms)
* `#[Medium]`: integration tests (under 1s)
* `#[Large]`: end to end tests (over 1s)
* `#[Group('wip')]`: arbitrary categories

Data providers:

* `#[DataProvider('methodName')]`: for a static method in the test class
* `#[DataProviderExternal(ClassName::class, 'methodName')]`: for a static method in a different class
* `#[TestWith([data])]`: inline, without declaring a static method

## Factory methods

Prefer private factory methods over `setUp()` to create the System Under Test (SUT):

* `setUp()` runs before every test, even those that don't need it
* test class properties, once instantiated, are kept in memory until the end of the testsuite run
* factory methods use local variables that are freed after each test

> **⚠️ PHPUnit creates one instance of each test class per test method and per data provider row,
> and keeps them all in memory until the testsuite completes.**

```php
#[CoversClass(Username::class)]
#[Small]
final class UsernameTest extends TestCase
{
    private function username(string $value = 'Merlin'): Username
    {
        return Username::fromString($value);
    }

    #[TestDox('It can be converted from/to string')]
    public function test_it_can_be_converted_from_and_to_string(): void
    {
        $this->assertSame('Merlin', $this->username()->toString());
    }

    #[DataProvider('invalidUsernameProvider')]
    #[TestDox('It fails when raw username $scenario')]
    public function test_it_fails_when_raw_username_is_invalid(
        string $scenario,
        string $invalidUsername,
    ): void {
        $this->expectException(ValidationFailedException::class);
        $this->username($invalidUsername);
    }

    /**
     * @return \Iterator<array{
     *     scenario: string,
     *     invalidUsername: string,
     * }>
     */
    public static function invalidUsernameProvider(): \Iterator
    {
        yield ['scenario' => 'is empty', 'invalidUsername' => ''];
        yield ['scenario' => 'is too short (< 4 characters)', 'invalidUsername' => 'abc'];
        yield ['scenario' => 'is too long (> 15 characters)', 'invalidUsername' => 'abcdefghijklmnop'];
    }
}
```

Output with `--testdox`:

```
Username
 ✔ It can be converted from/to string
 ✔ It fails when raw username is empty
 ✔ It fails when raw username is too short (< 4 characters)
 ✔ It fails when raw username is too long (> 15 characters)
```

## Test doubles

DTOs and Value Objects don't need test doubles: they're immutable data containers,
use real instances instead.

### Prophecy

When the SUT creates values internally, use Argument matchers:

* `Argument::type(ClassName::class)`: matches by type
* `Argument::that(static fn ($v): bool => ...)`: matches with a custom closure
* `Argument::any()`: matches anything

```php
use Prophecy\Argument;
use Prophecy\PhpUnit\ProphecyTrait;

#[CoversClass(SignInPlayerHandler::class)]
#[Small]
final class SignInPlayerHandlerTest extends TestCase
{
    use ProphecyTrait;

    public function test_it_signs_in_player(): void
    {
        $username = UsernameFixture::makeString();
        $player = PlayerFixture::make();

        // Stub: configure return value
        $findPlayer = $this->prophesize(FindPlayer::class);
        $findPlayer->find(
            Argument::that(static fn (Username $u): bool => $u->toString() === $username),
        )->willReturn($player);

        // Mock: assert it gets called
        $saveAuthToken = $this->prophesize(SaveAuthToken::class);
        $saveAuthToken->save(Argument::type(AuthToken::class))
            ->shouldBeCalled();

        $signInPlayerHandler = new SignInPlayerHandler(
            $findPlayer->reveal(),
            $saveAuthToken->reveal(),
        );
        $signedInPlayer = $signInPlayerHandler->run(new SignInPlayer(
            $username,
        ));

        $this->assertInstanceOf(SignedInPlayer::class, $signedInPlayer);
    }
}
```

## Smoke tests

For controllers and commands, craft the input, pass it to the application,
and verify the status code. Use `#[CoversNothing]` as smoke tests don't target a specific class:

```php
#[CoversNothing]
#[Medium]
final class SignUpNewPlayerControllerTest extends TestCase
{
    public function test_it_signs_up_a_new_player(): void
    {
        $appKernel = TestKernelSingleton::get()->appKernel();

        $request = Request::create(
            uri: '/api/v1/actions/sign-up-new-player',
            method: 'POST',
            server: ['CONTENT_TYPE' => 'application/json'],
            content: json_encode([
                'username' => UsernameFixture::makeString(),
                'password' => PasswordPlainFixture::makeString(),
            ], \JSON_THROW_ON_ERROR),
        );

        $response = $appKernel->handle($request);

        $this->assertSame(Response::HTTP_CREATED, $response->getStatusCode(), (string) $response->getContent());
    }
}
```

## CLI options

```console
phpunit

  # Selection:
  --group small                        Only run tests from the specified group(s)
  --covers 'Username'                  Only run tests that intend to cover <name>
  --filter 'UsernameTest'              Filter which tests to run
  --testsuite unit                     Only run tests from the specified testsuite(s)

  # Execution:
  --stop-on-failure                    Stop after first failure
  --order-by <order>                   default|defects|depends|duration|random|size

  # Reporting:
  --testdox                            Replace default result output with TestDox format
```

## Configuration

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="tests/bootstrap.php"

         cacheDirectory=".phpunit.cache"
         executionOrder="depends,defects"
         requireCoverageMetadata="true"
         beStrictAboutCoverageMetadata="true"
         beStrictAboutOutputDuringTests="true"
         displayDetailsOnPhpunitDeprecations="true"
         failOnPhpunitDeprecation="true"
         failOnRisky="true"
         failOnWarning="true"

         shortenArraysForExportThreshold="10"
         colors="true"
>
    <testsuites>
        <testsuite name="unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="integration">
            <directory>tests/Integration</directory>
        </testsuite>
    </testsuites>

    <source
        ignoreIndirectDeprecations="true"
        restrictNotices="true"
        restrictWarnings="true"
    >
        <include>
            <directory>src</directory>
        </include>
    </source>
</phpunit>
```

## Resources

* [So you think you know PHPUnit - Sebastian Bergmann - PHPDD2024](https://www.youtube.com/watch?v=qwRdnoeq1H8) ([article](https://phpunit.expert/articles/phpunit-features-that-surprise-even-professionals.html))
* [Optimizing Your Test Suite - Sebastian Bergmann - PHP fwdays 2021](https://www.youtube.com/watch?v=wR6YflVkAt4)
* [Sebastian's raytracer project](https://github.com/sebastianbergmann/raytracer/)
* [Testing with(out) dependencies - Sebastian Bergmann](https://www.youtube.com/watch?v=d3qXBEBNjHc) ([article](https://phpunit.expert/articles/testing-with-and-without-dependencies.html))
* [Testing with DTOs and Value Objects - Sebastian Bergmann](https://phpunit.expert/articles/testing-with-data-transfer-objects-and-value-objects.html)
* [How I manage test fixture - Sebastian Bergmann](https://phpunit.expert/articles/how-i-manage-test-fixture.html)
* [Do Not Mock What You Do Not Own - Sebastian Bergmann](https://thephp.cc/articles/do-not-mock-what-you-do-not-own)
* [PHPUnit documentation](https://phpunit.de/documentation.html)

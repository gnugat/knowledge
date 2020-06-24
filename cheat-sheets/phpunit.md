# PHPUnit cheat sheet

A highly opinionated [PHPUnit](http://phpunit.de/) cheat sheet.

* [Assertions](#assertions)
* [Annotations](#annotations)
* [Test doubles](#test-doubles)
    * [Stub](#stub)
    * [Mock](#mock)
* [Set up and tear down](#set-up-and-tear-down)
* [Usage](#usage)
* [Resources](#resources) 

## Assertions

Prototype: `self::assertX($expected, $actual, $message = '')`

* `assertSame`: strict equality (`===`)
* `assertEquals`: loose equality (`==`)
* `assertTrue`
* `assertFalse`
* `assertInstanceOf`

For exceptions:

* `$this->expectException(\Exception::class)`

## Annotations

* `@test`: to avoid `test` prefix on test methods
* `@group <name>`: to be used with `phpunit --group <name>`

## Test doubles

### Stub

```php
$stub = $this->prophesize(SomeClass::class);

$stub->doSomething()->willReturn('foo');
$stub->doSomething()->willThrow(\Exception::class);
```

### Mock

```php
$mock = $this->prophesize(SomeClass::class);

$mock->doSomething('something')->shouldBeCalled();
$mock->doSomething('something')->shouldBeCalledTimes(2);
```

## Set up and tear down

```php
<?php

class MyUnitTest extends PHPUnit_Framework_TestCase
{
    public static function setUpBeforeClass()
    {
        // Executed once before the first *test* method
    }
    
    protected function setUp()
    {
        // Executed before every *test* method
    }
    
    protected function tearDown()
    {
        // Executed after every *test* method
    }
    
    public static function tearDownBeforeClass()
    {
        // Executed once after the last *test* method
    }
}
```

## Usage

PHPUnit can be used for system tests, which allow you to check that an entry point of the application works as expected.

It can be achieved by:

1. defining an application input (Request, or Input)
2. asking the application to handle this input and retrieving its output (Response, exit code)
3. checking the status:
    * for web page, simply check that the response status code is 200
    * for APIs, check the exact status code
    * for CLI commands, check that the status code is 0

Its advantages:

* better coverage (checks that unit interacts as expected between each other)
* easy to write

Its drawbacks:

* usually slow to execute (can involve filesystem operations, database transactions and network requests)

### Example

Using Symfony and PHPUnit:

```php
<?php
 
namespace tests\AppBundle\Controller;

use Symfony\Component\HttpFoundation\Request;
 
class FortuneControllerTest extends \PHPUnit_Framework_TestCase
{
    private \AppKernel $app;

    protected function setUp()
    {
        $this->app = new \AppKernel('test', false);
        $this->app->boot();
    }
 
    /**
     * @test
     */
    public function it_lists_all_quotes()
    {
        $request = Request::create('/v1/fortunes', 'GET');
 
        $response = $this->app->handle($request);
        $this->app->terminate($request, $response);
 
        self::assertSame(200, $response->getStatuCode(), $response->getContent());
    }
}
```

And to test commands:

```php
<?php
 
namespace tests\AppBundle\Command;
 
use Symfony\Bundle\FrameworkBundle\Console\Application;
use Symfony\Component\Console\Tester\ApplicationTester;

class SubmitNewQuoteCommandTest extends \PHPUnit_Framework_TestCase
{
    private ApplicationTester $applicationTester;

    protected function __construct()
    {
        $kernel = new \AppKernel('test');
        $kernel->boot();
        $application = new Application($kernel);
        $this->applicationTester = new ApplicationTester($application);
    }
 
    /**
     * @test
     */
    public function it_submits_a_new_quote()
    {
        $exitCode = $this->applicationTester->run([
            'quote:submit',
            'quote' => 'Nobody expects the Spanish Inquisition!',
        ]);

        self::assertSame(0, $exitCode, $this->applicationTester->getDisplay());
    }
}
```

## Resources

* [Questioning PHPUnit best practices](https://thephp.cc/news/2016/02/questioning-phpunit-best-practices)
* [PHPUnit and prophecy](https://thephp.cc/news/2015/02/phpunit-4-5-and-prophecy)

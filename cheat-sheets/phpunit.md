# PHPUnit cheat sheet

A highly opinionated [PHPUnit](http://phpunit.de/) cheat sheet.

* [Assertions](#assertions)
* [Annotations](#annotations)
* [Test doubles](#test-doubles)
    * [Stub](#stub)
    * [Mock](#mock)
* [Set up and tear down](#set-up-and-tear-down)
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

## Resources

* [phpunit-accelerator](https://github.com/mybuilder/phpunit-accelerator)
* [Questioning PHPUnit best practices](https://thephp.cc/news/2016/02/questioning-phpunit-best-practices)
* [PHPUnit and prophecy](https://thephp.cc/news/2015/02/phpunit-4-5-and-prophecy)

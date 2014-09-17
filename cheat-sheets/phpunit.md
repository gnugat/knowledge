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

Prototype: `assertX($expected, $actual, $message = '')`

* `assertSame`: strict equality (`===`)
* `assertEquals`: loose equality (`==`)
* `assertTrue`
* `assertFalse`
* `assertInstanceOf`

## Annotations

* `@dataProvider <methodName>`
* `@expectedException <exceptionFcqn>`
* `@group <name>`: to be used with `phpunit --group <name>`

## Test doubles

### Stub

```php
$stub = $this->getMock('SomeClass');

// or
$stub = $this->getMockBuilder('SomeClass')
    ->disableOriginalConstructor()
    ->getMock()
;

$stub->method('doSomething')
    ->willReturn('foo')
;
$stub->method('doSomething')
    ->will($this->throwException(new \Exception()))
;
```

### Mock

```php
$mock = $this->getMock('SomeClass', array('doSomething'));

// or
$mock = $this->getMockBuilder('SomeClass')
    ->disableOriginalConstructor()
    ->setMethods(array('doSomething'))
    ->getMock()
;

$mock->expects($this->once())
    ->method('doSomething')
    ->with($this->equalTo('something'))
;
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

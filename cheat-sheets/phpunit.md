# PHPUnit cheat sheet

A highly opinionated [PHPUnit](http://phpunit.de/) cheat sheet.

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

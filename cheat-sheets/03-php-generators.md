# PHP Generators

A highly opinionated [PHP generator](http://fr2.php.net/manual/en/generator.valid.php) cheat sheet:

* [General](#general)
* [Generator class](#generator-class)
* [Links to use cases](#links-to-use-cases)
* [Examples](#examples) :
  * [simple example](#simple-example)
  * [range example](#range-example)
  * [co-routine example](#co-routine-example)

## General

In PHP ~5.5, a function containing `yield` becomes an instance of the
[`Generator` class](http://php.net/manual/en/class.generator.php), which acts like an iterator.

Each `yield` occurence marks a position. Calling `next` will move from the current yield to the next one.

Thanks to the `send` method, the `Generator` class acts also as a corroutine: you can send the returned value of the
`yield` occurence.

A `Generator` function cannot return a value.

## Generator class

```php
class Generator
{
    public function current(); // The yielded value.
    public function key();
    
    public function valid(); // Has the iterator been closed?
    
    public function next(); // Resumes execution.
    public function rewind();
    
    public function send($value); // Sends the value to be returned by the current yield, and resumes execution.
    public function throw(Exception $e); // Replaces the current yield with `throw $e;`
}
```

## Links to use cases

* [co-routines as an alternative to state machines](http://eli.thegreenplace.net/2009/08/29/co-routines-as-an-alternative-to-state-machines/)
* [Cooperative multitasking using co-routines in PHP](http://nikic.github.io/2012/12/22/Cooperative-multitasking-using-coroutines-in-PHP.html)
* [Python generators and co-routines](http://stackoverflow.com/a/1347772)
* [What generators can do for you](http://blog.ircmaxell.com/2012/07/what-generators-can-do-for-you.html)

## Examples

### Simple example

```php
function direct_generator()
{
    yield 1;
    yield 'index' => 2;
}

$generator = direct_generator();
echo $generator->current(); // outputs 1
$generator->next();
echo $generator->current(); // outputs 2
echo $generator->key(); // outputs index

```

### Range example

Foreach internally calls `current`, `key`, `next` and `valid` methods.

```php
function xrange($start, $end)
{
    $value = $start;
    while ($value <= $end) {
        yield $value;
        $value++;
    }
}

foreach(xrange(1, 2) as $value) {
    echo $value; // outputs 1, then 2.
}

```

### Co-routine example

Calling the `send` method is like replacing the current `yield` with the given value.

```php
function holy_grail_countdown($number)
{
    while (0 < $number) {
        $reset = (yield $number);
        $number = $reset ?: $number - 1; 
    }
}

$c = holy_grail_countdown(5);
foreach ($c as $number) {
    echo "$number\n"; // outputs 5, then 2 and finally 1.
    if (5 === $number) {
        $c->send(3);
    }
}
```

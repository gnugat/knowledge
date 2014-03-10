# PHP Generators

A highly opinionated [PHP generator](http://fr2.php.net/manual/en/generator.valid.php) cheat sheet.

## General

In PHP ~5.5, a function containing `yield` becomes an instance of the
[`Generator` class](http://php.net/manual/en/class.generator.php).

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

## Example

```php
function foo()
{
    $foo = (yield 'strlen' => ['a']);
 
    $foo .= (yield 'strlen' => ['ab']);
 
    $foo .= (yield 'strlen' => ['abc']);
 
    yield $foo."final";
}
 
function call_generator_function($g)
{
    $args = $g->current();
    $callable = $g->key();
 
    while ($g->valid() && is_callable($callable)) {
        $return = call_user_func_array($callable, $args);
        $g->send($return);
 
        $args = $g->current();
        $callable = $g->key();
    }
 
    return $args;
}
 
$result = call_generator_function(foo());
 
var_dump($result); // Outputs 'string(8) "123final"'
```

Reference: [call_generator_function, by Grégoire Pineau](https://gist.github.com/lyrixx/9b12efb7af8d51b1e269)

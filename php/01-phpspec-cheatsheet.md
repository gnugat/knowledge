# phpspec cheatsheet

Links:

* [phpspec official website](http://www.phpspec.net/)
* [prophecy repo](https://github.com/phpspec/prophecy)
* [introducing BDD, by Dan North](http://dannorth.net/introducing-bdd/)
* [phpspec illustrated by examples](http://www.slideshare.net/marcello.duarte/phpspec-20-ilustrated-by-examples)
* [emergent design with phpspec](http://www.slideshare.net/marcello.duarte/emergent-design-with-phpspec)

## Stub

Specifies collaborators return value. Used together with methods that return value:

    function it_can_be_assigned_tasks(Task $task)
    {
        $task->getCost()->willReturn(42);
        $this->assignTask($task);
    }

## Mock

Checks if collaborators methods have been called. Used together with methods that do not return value:

    function it_can_be_assigned_tasks(Task $task)
    {
        $task->changeCost(12)->shouldBeCalled();
        $this->changeTaskCost($task, 12);
    }

## Setup and teardown

    const RESTAURANT = 'Chli-Pou-Ni';

    function let(Food $food)
    {
        $this->beConstructedWith(self::RESTAURANT, $food);
    }

    function letGo()
    {
        // ??? - profit!
    }

## Formatters

`phpspec [--format=<name>|-f<name>]`:

* progress (default)
* dot (phpunit like)
* pretty (phpunit testdox)
* nyan (is it really working?)

## Matchers

Begin with `should` or `shouldNot`.

### Identity (`===`)

    $this->method()->shouldBe($value);
    $this->method()->shouldBeEqualTo($value);
    $this->method()->shouldReturn($value);

### Comparison (`==`)

    $this->method()->shouldBeLike($value);

### Exception

    $this->shouldThrow('\Exception')->duringMethod($name);
    $this->shouldThrow('\Exception')->during('method', array($name));
    $this->shouldThrow(new \Exception('message'))->during('method', array($name));

### Type

    $this->shouldHaveType('Class');
    $this->shouldReturnAnInstanceOf('Class');
    $this->shouldBeAnInstanceOf('Class');
    $this->shouldImplement('Class');

### State methods (`isXxx`, `hasXxx`)

    $this->shouldBeState();
    $this->shouldHaveState();

### Count

    $this->method()->shouldHaveCount($count);

### Primitive types (`is_*`)

    $this->method()->shouldBeString();
    $this->method()->shouldBeInteger();
    $this->method()->shouldBeArray();
    $this->method()->shouldBeBool();
    $this->method()->shouldBeDecimal();

### Custom

    public function getMatchers()
    {
        return [
            'haveKey' => function($subject, $key) {
                return array_key_exists($key, $subject);
            },
            'haveValue' => function($subject, $value) {
                return in_array($value, $subject);
            },
        ];
    }

## Configuration samples

### Project's composer.json

    {
        "require-dev": {
            "phpspec/phpspec": "~2.0@beta"
        },
        "autoload": {
            "psr-0": {
                "": "src"
            }
        }
    }

### Bundle's composer.json

    {
        "require-dev": {
            "phpspec/phpspec": "~2.0@beta"
        },
        "autoload": {
            "psr-0": {
                "Vendor\\Bundle\\NameBundle": ""
            }
        },
        "target-dir": "Vendor/Bundle/NameBundle"
    }

## FAQ

### phpspec wants to generate a class which already exists

check your autoloading

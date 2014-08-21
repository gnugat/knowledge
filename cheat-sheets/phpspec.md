# phpspec cheatsheet

A highly opinionated [phpspec](http://www.phpspec.net/) cheat sheet.

* [Stub](#stub)
* [Mock](#mock)
* [Setup and teardown](#setup-and-teardown)
* [Matchers](#matchers)
* [Resources](#resources)

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

## Matchers

Begin with

To prefix with `should` or `shouldNot`:

* `Be`: strict comparison (`===`)
* `BeLike`: loose comparison (`==`)
* `Implement` or `BeAnInstanceOf`
* `HaveCount`
* primitives:
    * `BeString`
    * `BeInteger`
    * `BeArray`
    * `BeBool`
    * `BeDecimal`

### Exception

    $this->shouldThrow('\Exception')->duringMethod($argument);
    $this->shouldThrow(new \Exception('message'))->duringMethod(array($argument));

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

## Resources

* [prophecy repo](https://github.com/phpspec/prophecy)
* [introducing BDD, by Dan North](http://dannorth.net/introducing-bdd/)
* [phpspec illustrated by examples](http://www.slideshare.net/marcello.duarte/phpspec-20-ilustrated-by-examples)
* [emergent design with phpspec](http://www.slideshare.net/marcello.duarte/emergent-design-with-phpspec)

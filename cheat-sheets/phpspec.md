# phpspec cheatsheet

A highly opinionated [phpspec](http://www.phpspec.net/) cheat sheet.

* [Stub](#stub)
* [Mock](#mock)
* [Setup and teardown](#setup-and-teardown)
* [Matchers](#matchers)
* [Resources](#resources)

## Stub

To use when a collaborator's method returns a value:

    function it_can_be_assigned_tasks(Task $task)
    {
        $task->getCost()->willReturn(42);
        $this->assignTask($task);
    }

## Mock

To use when collaborator's method doesn't return a value:

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
        $this->beConstructThrough('make', array(self::RESTAURANT, $food)); // Static constructor
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
* `Implement`, `BeAnInstanceOf` or `HaveType`
* `HaveCount`
* primitives:
    * `BeString`
    * `BeInteger`
    * `BeArray`
    * `BeBool`
    * `BeDecimal`

### Exception

    $collaborator->methodCall()->willThrow('\Exception');

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

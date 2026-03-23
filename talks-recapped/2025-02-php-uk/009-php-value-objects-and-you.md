# PHP Value Objects and You

> **Talk Abstract**
>
> Value Objects can be one of the most powerful tools in your modelling toolbox.
> They make code expressive, maintainable, deterministic and cohesive.
> Yet they are sometimes misunderstood and it's often difficult for developers to
> realise where and how they can use them.
>
> In this talk you will learn how to identify value objects in the wild,
> what their properties are and how to create your own!

Speaker: Dan Leech
* [Video](https://www.youtube.com/watch?v=woNLYVamn3A)
* [Blog post](https://www.dantleech.com/blog/2024/11/24/php-value-objects-and-you/)

## Recap

Dan Leech opens with an origin story: he wrote a blog post about value objects
after sitting in a meeting where colleagues debated which interfaces to implement,
which abstract classes to extend, and which Composer libraries to pull in before
they could start using value objects. His reaction was quiet frustration.

He was already using value objects in their project, without any of those things,
and found it hard to imagine working without them.

### What a value object is

The framing Dan reaches for is Michelangelo and the block of marble.
The sculptor does not impose a form on the stone.
He chips away the excess until the form that was already inside is revealed.
Programming works the same way: a problem arrives as a vague requirement, and the
model emerges as you work on it.
Value objects are one of the tools for that process.

A value is something whose identity is the sum of its parts.
The number seven is a value. You cannot update seven to become eight.
Eight is a separate value. An IP address works the same way: change any of its
components and you have a different IP address, a different identity.
An order entity, by contrast, has an identifier. Its status and contents can
change while it remains the same order. That distinction, identity by attributes
versus identity by a separate key, separates value objects from entities.

Value objects can be found in the wild as soon as you start looking:

```php
Date::fromYmd(2011, 1, 1)                  // a specific day in the year
Money::fromCentsAndCode(100, 'GBP')        // an amount of currency
Geolocation::fromLatLong(50.8137, -2.4747) // the exact position of the Cerne Abbas Giant
Color::fromRgb(165, 42, 42)                // the colour brown
ByteOffsetRange::fromInts(1, 2)            // a range of byte offsets
```

The minimal value object is just a class wrapping its attributes.
Dan's example is a color with red, green, and blue integer.
That alone is already ten times better than passing the three variables around separately.

### Building a good value object

**Immutability** comes first.
If you hold a reference to a value object and something elsewhere mutates it,
the value changes for everyone holding that reference.
Operations on a value object should return a new instance rather than modifying
the existing one.
The class should also be `final`: there is no good reason to extend a value object.

**Validation in the constructor** is what gives a value object its guarantee.
If a value object can be instantiated, it should be assumed valid.
Any function receiving a value object as a parameter does not need to
re-validate its contents:

```php
final readonly class Color {
    private function __construct(private int $r, private int $g, private int $b) {
        if ($r < 0 || $g < 0 || $b < 0 || $r > 255 || $g > 255 || $b > 255) {
            throw new RuntimeException(sprintf(
                'Invalid RGB value %d, %d, %d. All values must be between 0 and 255',
                $r, $g, $b
            ));
        }
    }

    public static function fromRgb(int $r, int $g, int $b): self {
        return new self($r, $g, $b);
    }
}
```

The validation lives in the constructor and every static constructor delegates to it.
This is the essential rule: no matter which format the value object is created from,
it is always validated by the same rules.
The constructor is the guard against invalid state.
Dan recommends against using an assertion library for this: coupling to an external
library to avoid a few lines of code also means the exceptions thrown are not part
of your domain, and good exception messages are worth writing by hand.

**Static constructors** are a natural next step for richer types.
Making the constructor private and exposing named static methods like `fromRGB`,
`fromHex`, or `fromHSV` does two things: it hides the internal representation
(which can then be changed without affecting callers), and it makes the available
entry points discoverable.
Typing `Color::` in an IDE shows every way the concept can be constructed.

No matter how the object was instantiated, equivalent values produce the same
internal state:

```php
$c1 = Color::fromRgb(154, 42, 42);
$c2 = Color::fromHex('#A52A2A');

assert($c1 == $c2); // true: they are the same value
```

The same pattern works in reverse.
`toRGB`, `toHSV`, `toColorName` turn the value object into a conversion pivot
between representations.
Ideally the transformation is lossless:

```php
Color::fromHex('#A52A2A')->toRgb() === [154, 42, 42];
Distance::fromKilometers(5)->toMiles() === 3.10686;
```

But lossy transformations are useful too:

```php
ClassName::from('Acme\\Baz\\Foobar')->namespace() === 'Acme\\Baz';
Date::fromYmd(2024, 01, 01)->dayOfTheYear() === 1;
```

**Composition** extends this further.
A `ByteOffset` value object can be combined with another to form a `ByteOffsetRange`.
The range gains its own behaviors while the constituent atoms remain reusable on
their own.

### Finding value objects in existing code

Dan's preferred signal is the utility class.
A `ColorUtils` class with static methods that take seven parameters
(R1, G1, B1, R2, G2, B2, plus a position) is a function that does not know
where it belongs.
Introducing a `Color` value object reduces the parameter count.
Introducing a `ColorRange` reduces it further.
Moving the interpolation method onto `ColorRange` eliminates the utility class
entirely.
The function found its home.

Another signal is a pair of variables that always travel together:

```php
// before
function make_payment(int $amount, string $code): Receipt;

// after
function make_payment(Money $money): Receipt;
```

As separate arguments, `$amount` and `$code` represent a quantity and a unit.
Together they represent an amount of currency, which is what the function actually needs.
Keeping them separate means they can diverge anywhere along a 1,000-line action method.
Wrapping them in a `Money` value object locks them together at the point of
construction and protects them for the rest of their journey through the code.
Dan's point is that this does not happen by design up front.
It emerges from iterating on the model, chipping away until the form reveals itself.

### Operators and equality

PHP's `==` operator compares object attributes by value, which means basic equality
between value objects works out of the box.
But equality is contextual.
Consider two polylines with the same set of coordinates in a different order:

```php
$polyline1 = [[0,0], [3,3], [3,0], [0,0]];
$polyline2 = [[0,0], [3,0], [0,0], [3,3]];
```

These render differently.
For a polyline, order matters. For a set of tags, it does not.
A value object is the right place to encode that distinction through an explicit
`equals()` method rather than relying on the generic `==` operator.

Greater-than and multiplication do not map cleanly to PHP operators, but that is fine.
The language of the domain is often more specific and more meaningful than generic symbols.
A color is not simply "greater than" another color.
It might be brighter, darker, or more saturated.
Domain-specific method names communicate intent in a way that overloaded operators
never could:

```php
$newColor = $color1->mix($color2);
$isBrighter = $color1->isBrighterThan($color2);
$date2 = $date1->addDays(2);
```

### Anti-patterns

**Bypassing the constructor in static methods.**
Assigning directly to private properties from within a static constructor rather
than delegating to the real constructor multiplies the places where validation
must happen.
Every static constructor becomes responsible for its own validation, and the
guarantee that "if it exists, it is valid" erodes.
Always use the constructor.
It is the single point of entry and the right place for the guard.

**Value object interfaces and abstract base classes.**
An interface that requires every value object to implement `equals`,
`greaterThan`, and `lessThan` imposes constraints that do not make sense for
all values.
Can a `ClassName` be said to be "greater than" another `ClassName`?
The interface reduces expressiveness without adding safety.
Value objects should start life in ignorance and evolve what they need.
Preemptively reaching for `extends` or `implements` is a code smell.

**Organizing by technical type rather than domain.**
A `ValueObjects` namespace that collects every value object in the project
regardless of domain is how legacy codebases start.
Organizing by bounded context instead gives each domain its own space and keeps
related concepts together:

```
src/
  Runner/
  Users/
  Charts/
    Color.php
    Gradient.php
    Gradients.php
```

**The `ValueObject` suffix.**
It does not matter that `Color` is a value object. It matters that it is a color.
The real world does not care about value objects, collections, or interfaces.
These are a consequence of modelling, not the goal of it.
`Color::fromRgb(255, 10, 255)` reads better than
`ColorValueObject::fromRgb(255, 10, 255)` in every context.

**Manual `fromArray` and `toArray` methods.**
How a value is represented on the wire is not the concern of the value itself.
Doing this by hand means the developer is responsible for validating that array
keys exist and have the right types, and developers are not good at that.
The property names end up referenced in multiple places.
None of that code is necessary if you use a mapping library, and you absolutely should.
Dan points to Valinor as an example worth looking at.

**Mocking value objects in tests.**
Neither side of the mocking debate advocates for mocking value objects.
They have no side effects, no dependencies, and no behavior that varies at runtime.
Mocking a value object is like mocking the number seven.
Instantiate it directly.

### The Q&A

On whether to throw domain-specific exceptions when construction fails: yes,
if it makes sense in the domain and the exception is likely to be caught.
For exceptions that will never be caught, a plain runtime exception is sufficient.
A follow-up from the audience noted that older PHP versions (before 7.4) could not
throw exceptions in certain magic method contexts, which is one historical
justification for static constructors that perform the validation before delegating.

On reusing a value object across domains: if two domains genuinely model the same
concept in the same way, extract it into a shared agnostic space between them.
If the needs diverge even subtly, separate objects serve the domains better.

On whether to use a validation library inside a value object: no.
Validation of user input belongs on the DTO at the controller boundary.
Once that input has been validated, it maps to value objects through the constructor,
which handles the domain-level constraints.
The two layers of validation serve different purposes and should stay separate.

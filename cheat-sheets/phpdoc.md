# phpdoc cheat sheet

A highly opinionated [phpdoc](http://www.phpdoc.org/) cheat sheet.

* [Types](#types)
* [Tags](#tags)
* [Inheritance](#inheritance)

## Types

* `int`
* `bool`
* `float`
* `<type>[]`

## Tags

* `@api`
* `@deprecated <version> <description>`
* `@todo [description]`
* `@var <type> [<name>] [<description>]`
* `@param <type> <name> [<description>]`
* `@return <type> [<description>]`
* `@throws <type> [<description>]`
* `@see <URI>|<FQSEN> [<description>]`

**FQSEN**: Fully Qualified Structural Element Name (e.g. `\My\Space\MyClass::myMethod()`).

## Inheritance

Elements automatically inherit from their parent's phpdoc (no need for `{@inheritDoc}`).

To include a parent's phpdoc inside a child phpdoc, use `{@inheritDoc}`:

```php
class Parent
{
    /**
     * Parent description.
     */
    public function foo();
}

class Child extends Parent
{
    /**
     * Child description (before). {@inheritDoc} Child description (after).
     */
    public function foo();
}

// Result: "Child description (before). Parent description. Child description (after)"
```

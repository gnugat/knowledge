# phpdoc cheat sheet

A highly opinionated [phpdoc](http://www.phpdoc.org/) cheat sheet.

* [Types](#types)
* [Tags](#tags)
* [Inheritance](#inheritance)

## Types

* `int`, `bool`, `float`, `string`
* `?type` or `type|null`: nullable (eg `?string`)
* `class-string<T>`: fully qualified class name (eg `class-string<\Throwable>`)
* `<type>[]`: array of given type (eg `string[]`)
* `list<type>`: sequential array indexed from 0 (eg `list<string>`)
* `array{key: type}`: array with specific named keys (eg `array{name: string, age: int}`)
* `list<array{key: type}>`: list of shaped arrays (eg `list<array{name: string, age: int}>`)
* `\Iterator<type>`: iterator (eg `\Iterator<array{name: string}>`)

Array shapes are useful to type SQL query results:

```php
/**
 * @var list<array{
 *     username: string,
 *     love_points: int,
 * }> $rows
 */
$rows = $stmt->fetchAll();
```

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

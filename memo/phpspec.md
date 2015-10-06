# phpspec

A [spec BDD](http://dannorth.net/introducing-bdd/) tool allowing you to write unit tests.
Its advantages:

* better TDD process (automates it by generating test and code boilerplates)
* better readability (explicit yet non verbose mocking library and assertions)
* better test practices (e.g. no private/protected/static tests)

Its drawbacks:

* doesn't support the following features, on purpose (see [My top 10 favourite phpspec limitations](http://techportal.inviqa.com/2014/09/11/my-top-ten-favourite-phpspec-limitations/)):
    * Cannot test private methods: make them public, and even better extract them into their own object
    * No code coverage: driving your development by test automatically covers the essentials parts
    * Separate code standard: test methods should be sentences
    * Cannot mock methods of the SUS: extract them into their own object

##  Test doubles

When we test a class (System Under Test, SUT), we often have to deal with dependencies (Collaborators).
phpspec allows us to describe how a SUT interacts with its Collaborators via test doubles:

* if the method called returns something, we can force it to return a given value (stubbing)
* else we can check if the method is called (mocking)

> **Note**: [phpspec](http://phpspec.net/) uses [prophecy](https://github.com/phpspec/prophecy) as a test double library.
> This same library is [now used natively in PHPUnit](https://phpunit.de/manual/4.5/en/test-doubles.html#test-doubles.prophecy) (4.5).

## Example

We want to create a service which formats durations. First we'll generate the test file:

    phpspec describe 'Vendor\Project\TextEditor'

> **Note**: We can shorten `describe` to `d`.

Then we write our first test:

```php
<?php
// File: spec/Vendor/Project/TextEditorSpec.php

namespace spec\Vendor\Project;

use Vendor\Project\Service\Filesystem;
use Vendor\Project\File;
use PhpSpec\ObjectBehavior;

class TextEditorSpec extends ObjectBehavior
{
    const FILENAME = '/tmp/file.txt';
    const FORCE_FILE_CREATION = true;

    function let(Filesystem $filesystem)
    {
        $this->beConstructedWith($filesystem);
    }

    function it_can_force_file_creation_if_it_does_not_already_exists(File $file, Filesystem $filesystem)
    {
        $filesystem->exists(self::FILENAME)->willReturn(false);
        $filesystem->create(self::FILENAME)->willReturn($file);

        $this->open(self::FILENAME, self::FORCE_FILE_CREATION)->shouldBe($file);
    }
}
```

Next we generate the code file:

    phpspec run

> **Note**: We can shorten `run` to `r`.

Now we need to write the code which will make the test pass:

```php
<?php

namespace Vendor\Project;

use Vendor\Project\Service\Filesystem;

class TextEditor
{
    private $filesystem;

    public function __construct(Filesystem $filesystem)
    {
        $this->filesystem = $filesystem;
    }

    public function open($filename, $forceCreation = false)
    {
        if (false === $this->filesystem->exists($filename) && true === $forceCreation) {
            return $this->filesystem->create($filename);
        }
    }
}
```

Finally we make sure that the test actually pass:

    phpspec r

All green! We can now add more tests to refine TextEditor's behavior (e.g. when a file already exists),
and then write more code, etc...

## Extensions

* generate typehints and meaningful names for arguments: https://github.com/memio/spec-gen

## Resources

* https://github.com/phpspec/prophecy
* http://dannorth.net/introducing-bdd/
* http://www.slideshare.net/marcello.duarte/phpspec-20-ilustrated-by-examples
* http://www.slideshare.net/marcello.duarte/emergent-design-with-phpspec
* http://techportal.inviqa.com/2014/09/11/my-top-ten-favourite-phpspec-limitations/
* http://www.slideshare.net/CiaranMcNulty/why-your-test-suite-sucks
* http://gnugat.github.io/2015/08/03/phpspec.html

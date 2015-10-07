# Symfony

## System test

A practice allowing you to check that an entry point of the application works as expected.

It can be achieved by:

1. defining an application input (Request, or Input)
2. asking the application to handle this input and retrieving its output (Response, exit code)
3. checking the status:
    * for web page, simply check that the response status code is 200
    * for APIs, check the exact status code
    * for CLI commands, check that the status code is 0

Its advantages:

* better coverage (checks that unit interacts as expected between each other)
* easy to write

Its drawbacks:

* usually slow to execute (can involve filesystem operations, database transactions and network requests)

### Example

Using Symfony and PHPUnit:

```php
<?php
 
namespace tests\AppBundle\Controller;

use Symfony\Component\HttpFoundation\Request;
 
class FortuneControllerTest extends \PHPUnit_Framework_TestCase
{
    protected function setUp()
    {
        $this->app = new \AppKernel('test', false);
        $this->app->boot();
    }
 
    /**
     * @test
     */
    public function it_lists_all_quotes()
    {
        $request = Request::create('/v1/fortunes', 'GET');
 
        $response = $this->app->handle($request);
 
        self::assertSame(200, $response->getStatuCode(), $response->getContent());
    }
}
```

And to test commands:

```php
<?php
 
namespace tests\AppBundle\Command;
 
use Symfony\Bundle\FrameworkBundle\Console\Application;
use Symfony\Component\Console\Tester\ApplicationTester;

class SubmitNewQuoteCommandTest extends \PHPUnit_Framework_TestCase
{
    private $applicationTester;

    protected function __construct()
    {
        $kernel = new \AppKernel('test');
        $kernel->boot();
        $application = new Application($kernel);
        $this->applicationTester = new ApplicationTester($application);
    }
 
    /**
     * @test
     */
    public function it_submits_a_new_quote()
    {
        $exitCode = $this->applicationTester->run(array(
            'quote:submit',
            'quote' => 'Nobody expects the Spanish Inquisition!',
        ));

        self::assertSame(0, $exitCode, $this->applicationTester->getDisplay());
    }
}

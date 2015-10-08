# CommandBus

A pattern allowing you to remove the logic from controllers, for imperative actions (create, update, delete, etc).
It is composed of 3 objects:

* **Command**: encapsulates the user's will, its purpose is to make the action's intention explicit (via its name)
  and to validate basic user input
* **CommandHandler**: does the actual logic. A CommandHandler manages only one Command
* **CommandBus**: responsible of finding the appropriate CommandHandler for the given Command

Its advantages:

* better testability (CommandHandler and Command can be unit tested)
* better readability (Command's name communicate the intent)
* better reusability (the same logic can be used in a controller, a command, an event listener, etc)

Its drawbacks:

* harder to find the chain of execution (have to know the convention or read the configuration to know which handler will be called by CommandBus)
* steep learning curve (new way of thinking: CommandBus doesn't return anything)

> **Note**: Command Bus allows to implement part of the Command / Query Separation Responsibility (CQRS)
> and is often used in combination with Event Sourcing

## Example

Using [Symfony](https://symfony.com/) and [Tactician](https://tactician.thephpleague.com/), the controller:

```php
<?php
 
namespace AppBundle\Controller;
 
use Acme\Fortune\Quote\SubmitNewQuote;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Method;
use Symfony\Component\Routing\Annotation\Route;
 
class QuoteController extends Controller
{
    /**
     * @Route("/fortunes")
     * @Method({"POST"})
     */
    public function submitNewAction(Request $request)
    {
        $submitNewQuote = new SubmitNewQuote(
            $request->query->get('quote')
        );
 
        $this->get('tactician.commandbus')->handle($submitNewQuote);
 
        return new Response('Quote submitted', 201);
    }
}
```

The command:

```php
<?php
 
namespace Acme\Fortune\Quote;

class SubmitNewQuote
{
    public $quote;
 
    public function__construct($quote)
    {
        if (null === $quote) {
            throw new \InvalidArgumentException('The mandatory "quote" parameter was missing');
        }
        $this->quote = $quote;
    }
}
```

The CommandHandler:

```php
<?php
 
namespace Acme\Fortune\Quote;

use Acme\Fortune\Quote\Service\CheckQuoteDuplicates;
use Acme\Fortune\Quote\Service\SaveNewQuote;
 
class SubmitNewQuoteHandler
{
    private $checkQuoteDuplicates;
    private $saveNewQuote;
 
    public function __construct(CheckQuoteDuplicates $checkQuoteDuplicates, Profilerepository $saveNewQuote)
    {
        $this->checkQuoteDuplicates = $checkQuoteDuplicates;
        $this->saveNewQuote = $saveNewQuote;
    }
 
    public function handle(SubmitNewQuote $submitNewQuote)
    {
        $quote = $submitNewQuote->quote;
        if (true === $this->checkQuoteDuplicates->hasDuplicates($quote)) {
            throw new \DomainException("Quote \"$quote\" already exists and duplicates are not allowed");
        }
        $this->saveNewQuote->save(array(
            'quote' => $quote,
        ));
    }
}
```

Exceptions can be handled in an event listener, not shown to make the example shorter.

## Nice resources

* http://shawnmc.cool/command-bus
* http://www.reddit.com/r/PHP/comments/29a6qz/what_am_i_missing_with_this_whole_command_bus/
* Command Bus series:
    * Command and CommandHandler: http://php-and-symfony.matthiasnoback.nl/2015/01/a-wave-of-command-buses/
    * CommandBus: http://php-and-symfony.matthiasnoback.nl/2015/01/responsibilities-of-the-command-bus/
    * EventBus: http://php-and-symfony.matthiasnoback.nl/2015/01/from-commands-to-events/
    * FAQ: http://php-and-symfony.matthiasnoback.nl/2015/01/some-questions-about-the-command-bus/
    * Event dispatching: http://php-and-symfony.matthiasnoback.nl/2015/01/collecting-events-and-the-events-aware-command-bus/
* http://verraes.net/2015/01/messaging-flavours/
* http://verraes.net/2015/02/form-command-model-validation/
* https://speakerdeck.com/richardmiller/avoiding-the-mud
* http://benjamindulau.com/blog/posts/do-not-mistake-ddd-for-cqrs-yeah-but-where-to-start
* http://verraes.net/2014/05/functional-foundation-for-cqrs-event-sourcing/
* http://www.udidahan.com/2009/12/09/clarified-cqrs/
* http://martinfowler.com/bliki/CQRS.html

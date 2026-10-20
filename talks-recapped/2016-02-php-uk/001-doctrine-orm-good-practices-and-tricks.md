# Doctrine ORM Good Practices and Tricks

> **Talk Abstract**
>
> Doctrine ORM is a complex tool that enables development of very advanced
> applications, but are you actually using it correctly?
> What if you have been using the ORM incorrectly or in an inefficient way?
> We are going to inspect the advantages and disadvantages of various
> approaches to application design with this Data mapper, and come up with a
> set of best practices that will be useful for your projects.

Speaker: Marco Pivetta (Ocramius)
* [Video](https://www.youtube.com/watch?v=rzGeNYC3oz0)
* [Slides](https://ocramius.github.io/doctrine-best-practices/)
* [🐙 github](https://github.com/ocramius)
* [🦣 mastodon](https://mastodon.social/@ocramius)

## Recap

Ocramius starts by reminding Doctrine is not just the ORM:
it is a group of persistence oriented libraries,
dealing with everything related to loading and saving stuff.

He also admits that he hates the ORM, 
which he reminds is mainly inspired by JPA (JSR 317) and Hibernate.

### Know your enemy

When starting with something new, it's easy to copy examples and use it everywhere.

But by reading the documentation, and event the code, you get a much better understanding,
including when NOT to use it.

### What Doctrine ORM is for

Doctrine is designed around **OLTP** (Online Transaction Processing):
which means many tiny transactions, each committed on its own (very CRUD like).

Ocramius recommends a DDD approach, which should be used as a tool for fast prototyping:
code objects, add use cases, and rely only on the database as a detail.

He argues that for apps that deal with one single database and need its specific feature,
Doctrine ORM is probably not the right tool.

The same goes if you do not want an object oriented design.

There are emore use cases where the ORM should not be used at all:

* **Dynamic data structures**: MongoDB style mixed data,
  or Magento's entity attribute value model (where a field is null, then zero, then a string)
  The ORM is about strict types and well defined structures.
* **Reporting**: this is where everyone complains about slowness,
  just use SQL!

### Entities

Entities should work without the ORM, and without the database:
they help you model your problem, they represent your domain,
so start by designing them before defining the database schema.

The ORM and the database are just about persistence.

### Entities are not typed arrays

An object with setters and getters is just an array with types,
a record that has no interraction with the real world.

Entities should not be like that, they define a behaviour,
create methods based on their interractions with other objects.

Ocramius highlights that starting by designing the state first leads to hidden coupling,
which inevitably freeze you into database mindset.

To find the interractions, it helps to write the test first.

A better `User` looks something like this:

```php
final class User
{
    // state skipped on purpose

    public function toNickname(): string
    { /* ... */ }

    public function authenticate(string $pass, callable $checkHash): bool
    { /* ... */ }

    public function changePass(string $pass, callable $hash)
    { /* ... */ }
}
```

* `toNickname()` is not really different from `getNickname()`, but it reads like a cast:
  we are transforming the object rather than fetching its state
* `authenticate()` and `changePass()` take a callable
  (for example something compatible with the `password_hash()` API)

We tell the object what to do,
and the object is free to reject the interaction by throwing an exception.

If your objects have no behaviour, you do not need an ORM:
it will cause more problems than it solves.

### Respect the Law of Demeter

Ocramius explains the Law of Demeter with Chinese whispers:
say "lion" at one end of the line and "snake" comes out at the other.

Only talk to your direct neighbours,
otherwise you end up with chained calls that look like a train wreck.

His example is a `User::hasAccessTo($resource)` method that:

* fetches the user's role
* gets its access levels
* assumes they come in a specific data structure
* loops over them to check access

That is 3 jumps through state.

Refactoring it so the user simply asks its role
(`$this->role->allowsAccessTo()`) moves the loop elsewhere,
but leaves a single level of indirection.

It is more expressive, easier to test and mock, less coupled, and easier to replace.

The same reasoning leads to **disallowing collection access from outside the entity**.

In the example,
a `banUser(Uuid $userId)` service method fetches the user from the repository,
calls `getBans()` and adds a ban to the returned collection.

This mutates state by reference, through several levels of indirection,
which is really hard to debug.

Instead:

* never expose the collection
* return a copy (a new collection, or `$collection->toArray()`)
* even better, turn it into an interaction: `$user->ban()`

### Entity validity

Ocramius admits that he used to validate entities.

But Entities should always be in a valid state after construction.

Data whose validity changes over time like a form being filled in,
belongs to a different concept: a data transfer object such as `UserFromInputForm`.

Once the DTO holds valid data, it can be turned into an entity.

Invalid data that reaches persistence is a moving bug:
you save an object, and one year later it blows up.

To avoid BC breaks on the constructor every time input requirements change, use
**named constructors**:

```php
public static function fromFormData(/* ... */): self
public static function fromSerializedState(/* ... */): self
public static function fromApiData(/* ... */): self
```

Since state stays valid and interactions replace state manipulation, **setters go away**:
a setter does not really mean anything anyway.

### Avoid coupling with the application layer

An application may live six months, or twenty years, and the framework may not be
supported by then.
Keep the framework out of the core logic.

Two bad examples:

* a `UserController::registerAction()` binding a form to `new User()`.
  Why can an empty user exist without an ID or a username?
  This breaks the valid-after-construct rule, and the form talks directly to the user's state.
  People then end up changing the entity to make the form work, adding getters and
  setters every time a field changes, instead of adapting the form to the entity.
* `User::fromFormData(Form $form)` is better, but the core logic now type hints
  against a framework class, so upgrading the framework means rewriting it.

Both Symfony Form and Zend Form break entity validity, with no exception.

The fix is again a DTO: bind the form to the DTO,
let it hold invalid state while fields are added and removed,
and create the entity from it once everything is valid.

### Lifecycle callbacks are a persistence hack

There are legitimate use cases:
think of lifecycle callbacks as `serialize()` and `unserialize()`,
but specific to the database.

For example, moving a big image or video to S3 instead of storing it in the database.

The problem is when people put business interactions inside them.

Would you implement `JsonSerializable` so that it sends an email every time the object is serialized?

That is exactly what business logic in a lifecycle callback does.

### Avoid auto-generated identifiers

An entity, by definition, has an identity.

With database generated identifiers, the entity has no ID until it has been saved,
which is an invalid state.

It also brings pitfalls:

* inserts block each other, since each one waits for the previous ID
* bulk inserts become impossible,
  as Doctrine must loop and fetch the last inserted ID to populate the object
* data appears by magic, which confuses newcomers
* multi-request transactions (create now, commit later) get complicated
* the object no longer works without the database, so the text file exercise fails

The simple solution is UUIDs, using Ben Ramsey's `ramsey/uuid` library:

```php
public function __construct()
{
    $this->id = Uuid::uuid4();
}
```

Ocramius designs all his new systems this way, with minimal overhead.

To give an idea of how random they are:
generating a million UUIDs per second for a hundred years gives roughly a 50% chance of a single collision.

The usual objection is that UUIDs cannot be sorted.

But **auto increment is often abused for sorting**:
what you are really looking for is a `DATETIME` field.

### Avoid derived and composite primary keys

A **derived primary key** is part of the primary key while also referencing another table.

In Doctrine terms: an `@Id` field that is also a to-one association.

Just don't: create a surrogate field, add a unique constraint for the rest,
and let it crash if duplicates hit the database.

You do not need everything in third normal form.

Ask whether the business actually needs that normalization,
or whether you are doing it because a book told you to.

The same goes for **composite primary keys**, spanning several columns.

This is not a hundred percent rule, but in most cases a self assigned ID
plus a unique constraint is simpler for fetching and saving,
and it makes no difference from a domain perspective.

### Favour immutable entities

Or at least **append-only data structures**.

In the example, a `PrivateMessage` has a sender, a recipient,
a message and a collection of who read it.

The `read()` interaction does not flip a boolean from false to true:
it appends a new `MessageRead` object,
which has no behaviour and can simply record when it was created.

The data grows over time, but disk space is cheap and databases are good at handling lots of rows.

As a side effect you get an audit log, and new business rules become possible:
how many times was it read, and when was it read first?

Immutable data is:

* **simple** to reason about
* **cacheable forever**, which pairs well with the Doctrine 2.5 second level cache:
  once cached, it never needs to be queried again
* **predictable**, you can see in which direction data is going
* **enabling historical analysis**

Taken further, this leads to event sourcing, which deserves its own talk.

### Avoid soft deletes

Soft deletes come from an age where everything had to live in one database,
with 700 tables, for performance reasons.

They break immutability, and worse, they break **data integrity**:
foreign keys now reference rows that are deleted but not really deleted.

That integrity is the very reason you picked an RDBMS over something like MongoDB.

Alternatives:

* implement an audit log, adding a record every time something is deleted
* ask whether anyone actually requested a "restore deleted data" feature
* or just restore a backup, which Ocramius says is perfectly fine

Most of the time, soft deletes can be replaced by a more specific domain concept.

Ocramius tells the story of a client whose metal pressing machines are each the size of the conference room.

After producing a hundred million pieces, a machine is boxed up and shipped elsewhere to make room for a new one.

It is not deleted.

They used soft deletes to switch machines on and off,
and ran into referential integrity issues when tracking which piece was produced by which machine.

Replacing it with an **archived** marker was a perfect fit.

### Mapping drivers

* **Annotations in private packages**: simple to read,
  any junior developer understands them (even if they will misconfigure them a couple of times),
  and they sit right next to the code they affect
* **XML in public packages**: users of your open source package can replace the mappings with their own
* **YAML**: don't

### Lazy or eager?

Do not decide this when first building the system:
don't mark everything as `EXTRA_LAZY` or `EAGER` by default, figure it out later.

**Eager loading is useless**:
it was meant to join associations automatically and fetch everything in one query,
but it is far from working that way.

The ORM is still a box of tricks, not magic.

**Extra lazy indicates high risk areas**:
you will need it sometimes, but it is a warning that you reached the limits of the ORM.

A collection with a thousand or a million items, looped over by accident,
can load the whole table into memory, swap, and take the system offline.

Be careful about performance and transaction size there,
and ask whether the ORM is still the right tool for that data.

### Avoid bi-directional associations

Straight from the manual: don't make associations bi-directional by default.

An object references its parts,
but the parts do not need to know what contains them.

Bi-directional associations are overhead,
as the ORM hydrates data in both directions.

Only code the associations that the business logic strictly needs.

If a complex DQL query needs to traverse associations in other directions,
hack it together, or use SQL.

### Repositories

**Custom repositories improve expressiveness**:
a method like `UserRepository::findUsersThatHaveAMonthlySubscription()` cannot be misread,
and behind it you can use SQL, DQL, a MongoDB MapReduce, caching, whatever you want.

Ocramius has lately preferred a more functional approach: **query functions**.

Move the method name to the class name, `UsersThatHaveAMonthlySubscription`,
which gets the entity manager injected and executes the query in `__invoke()`.

Opening the directory then lists every report your system supports.

**Repositories are services**, and so are query functions.

`$entityManager->getRepository()` is a service locator,
a utility to get started quickly, with all the usual service location problems.

Repositories are part of your domain logic:
**inject them in the constructor instead**.

**Separate `get` and `find` methods**:

* `find*()` may return `null` or an empty array
* `get*()` always returns something, or throws

```php
public function getBySlug(string $slug): BlogPost
{
    // ...
    throw BlogPostNotFoundException::bySlug($slug);
}
```

Consumers no longer have to deal with `null`.

A missing record crashes with a 500, which is fine for now,
and later you can catch the exception and turn it into a 404.

### Clear the entity manager between services

When different services deal with similar concepts and share state,
call `EntityManager::clear()` at the end of each piece of domain logic.

The identity map used to be the reason to keep entities in memory,
but with the second level cache, direct fetches mostly hit the cache anyway.

Instead of passing a `User` object from one service to the next, pass its identifier.

Across boundaries, like invoicing and shipping each having their own products,
pass product IDs rather than sharing the same entity.

This avoids shady transient state, and the recurring question of who is
responsible for flushing.

### Keep normalization (freaks) under control

Real world applications are not about a perfect schema that represents the whole world.

When you see a graph of 500 entities, sometimes the answer is just a JSON field stuffed with data.

You may need to gag your DBAs, or better, talk to them and get them to understand your needs.

This is not 70s style normalization on a single database,
and academic and practical knowledge differ:
a perfectly normalized schema can require very complicated queries,
while a denormalized one just works.

### What about performance?

Performance deserves its own talk, but here are the hotspots to profile:

* the **UnitOfWork**, where everything in memory lives
* **DQL**, and how it translates queries
* the **Paginator**: it is a monster,
  built for compatibility with Oracle and engines that do not support SQL 92.
  If it is slow, don't hack it, write a custom query
* **custom repositories**: instead of a very complicated query returning a
  thousand entities matching ten thousand business rules,
  fetch the identifiers first, then pass them to a query with an `IN` condition
* the **second level cache**: try it, it makes a big difference

Measuring is the only way: write the feature, then profile.

### Recap

* **Domain first**: think about what the business needs before picking tools.
  Doctrine may not be the right fit, and pure SQL applications are fine too.
* **Do not normalize without a need for it**, otherwise you are digging your own grave.
  Normalization must be a business requirement.
* **Consider using separate databases**: transactional data is not reporting data.
  Logs, copying data around, and backups are all valid.

### Q&A

**Should MongoDB style data be avoided?**

Not MongoDB itself: data that does not follow a predictable form should not go through the ORM.

A document store is perfectly fine for it,
but you then shift validation from database constraints to the application,
validating on every fetch and every store.

**What is the difference between the ObjectManager and the EntityManager?**

`ObjectManager` is the interface implemented by the `EntityManager`.

It was extracted into Doctrine Common as a minimal API shared by the different
persistence libraries to interact with a unit of work.

**If the ORM is not for reporting, how do I report on something computed, like a nickname?**

Copy the data into a reporting friendly structure, using batch processing,
listeners, or even database triggers.

Analytics and reporting are usually subsystems, not part of the core domain.

Build the application first, then the reporting subsystem, with its own schema,
queries and optimizations.

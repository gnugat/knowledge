# Object Oriented Programming (OOP)

The OOP paradigm is the fact of using exclusively objects to solve a problem.

## What is an object?

An object is a group of variables and functions, in order to define a behavior based on its data.

Vocabulary:

* object variables or functions: members
* object variables: attributes, properties
* object functions: methods
* calling an object's method: sending it a message
* code definition of the object: a class
* object created and stored in a variable: an instance

The object is responsible of manipulating its data, which means it should protect it against external changes.
To do so, you can use visibility on its members:

* public: the member can be called everywhere
* private: the member can only be called by the object itself

To initialize an object, a constructor must be defined.

Example:

    <?php
    
    class User
    {
        // Attributes
        private $firstName;
        private $lastName;
        
        // Constructor
        public function __construct($firstName, $lastName)
        {
            $this->firstName = $firstName;
            $this->lastName = $lastName;
        }
        
        // Method
        public function getFullName()
        {
            return $this->firstName.' '.$this->lastName;
        }
    }
    
    // Creating an instance of User
    $user = new User('Loïc', 'Chardonnet');

    // Print 'Loïc Chardonnet'
    echo $user->getFullName();

    // Will cause a fatal error (Cannot access private property)
    echo $user->firstName;

## Relationship

An object can own another one (or even a collection of another one), and it can also manipulate one.

Example:

    <?php
    
    class User
    {
        // Attributes
        private $firstName;
        private $lastName;
        
        // A collection of User
        private $children;
        
        // Constructor
        public function __construct($firstName, $lastName, array $children = array())
        {
            $this->firstName = $firstName;
            $this->lastName = $lastName;
            $this->children = $children;
        }
        
        public function makeChild($firstName)
        {
            $child = new User($firstName, $this->lastName);
            $this->children[] = $child;
            
            return $child;
        }
        
        // Method
        public function getFullName()
        {
            return $this->firstName.' '.$this->lastName;
        }
    }

### Tips

A private member can only be accessed by an instance of the object. Which means the following example is possible:

    <?php
    
    class User
    {
        // Attributes
        private $firstName;
        private $lastName;
        
        // Constructor
        public function __construct($firstName, $lastName)
        {
            $this->firstName = $firstName;
            $this->lastName = $lastName;
        }
        
        public function joinInWedlock(User $spouse)
        {
            $this->lastName = $spouse->lastName;
        }
        
        // Method
        public function getFullName()
        {
            return $this->firstName.' '.$this->lastName;
        }
    }

This behavior is not unique to PHP, for example it is the same as
[in C++](http://stackoverflow.com/questions/6921185/why-do-objects-of-the-same-class-have-access-to-each-others-private-data).

## What are the different kinds of objects?

According to [Anthony Ferrara](http://blog.ircmaxell.com/2013/11/beyond-object-oriented-programming.html), there are 5
main kinds of objects.

### Representers

Can also be called entities or domain models.
Its responsibility is to define the state of the data.

Example:

    <?php
    
    class BlogPost
    {
        public function isPublished() {}
        public function getTitle() {}
        public function getContent() {}
    }

#### Tips

* [some consider that representers shouldn't implement any behavior](http://www.slideshare.net/kriswallsmith/how-kris-writes-symfony-apps)
* [some consider that representers should implement a behaviour](http://www.martinfowler.com/bliki/AnemicDomainModel.html)

### Doers

Can also be called services.
Its responsibility is to execute a task, often by manipulating a representer.

Example:

    <?php
    
    class Logger
    {
        public function error($message) {}
    }

#### Tips

Doers are often stateless (do not have any attributes, or do not change its attributes values).

### Plumbers

Can also be called controllers.
Its responsibility is to pass an input representer to a doer, and  then to return an output representer.

Example:

    <?php
    
    class HttpKernel
    {
        public function handle(Request $request) {}
    }

#### Tips

Plumbers do nothing and should be kept short. It can follow these steps:

1. validating the input
2. processing it
3. building the output using:
   * the return value of the doer
   * the result of the validation

### Translators

Its responsibility is to take a representer and to transform it into another one.

Example:

    <?php
    
    class Yaml
    {
        /** @return PHP array */
        public function parse($fileContent) {}
    }

#### Tips

Can be a serializer, a converter, etc.

It does not correspond to the design pattern Adapter in which case the Adapter class takes a representer
and is used as the transformed one.

### Makers

Can also be called factories or builders.
Its responsibility is to create instances.

Example:

    <?php
    
    class UserFactory
    {
        /** @return an instance of User */
        public function make() {}
    }

#### Tips

Makers can create instances of representers, but also of translators, plumbers, doers or even other makers.

## References

* http://www.inf.ufsc.br/poo/smalltalk/ibm/tutorial/oop.html
* http://www.php.net/manual/en/language.oop5.php
* http://blog.ircmaxell.com/search/label/Beyond

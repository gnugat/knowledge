# Doctrine's cheat sheet

A highly opinionated [Doctrine](http://www.doctrine-project.org/) cheat sheet

## QueryBuilder

### Index by

```php
    $em
        ->createQueryBuilder()
        ->from('AcmeDemoBundle:Example', 'alias', 'alias.id')
    ;
```

Reference: [StackOverflow, via Grégoire Pineau and Jean François Simon](http://stackoverflow.com/a/15120793)

### Where in

```php
    $em
        ->where('alias.field IN (:parameter)')
        ->setParameter('parameter', array())
    ;
```

Reference: [StackOverflow](http://stackoverflow.com/a/11874278)

# Doctrine's cheat sheet

A highly opinionated [Doctrine](http://www.doctrine-project.org/) cheat sheet.

* [QueryBuilder](#querybuilder)
    * [Factories](#factories)
    * [Code example](#code-example)
* [EntityRepository](#entityrepository)
    * [Counting](#counting) 
* [Resources](#resources)

## QueryBuilder

Reference: [The QueryBuilder, high level API](http://docs.doctrine-project.org/en/latest/reference/query-builder.html#high-level-api-methods).

```php
namespace Doctrine\DBAL\Query;

class QueryBuilder
{
    public function select($select); // Overrrides all previously set conditions
    public function addSelect($select);

    public function where($predicates); // Overrrides all previously set conditions
    public function andWhere($predicates);
    
    public function setParameters(array $keyValues);
    
    public function orderBy($field, $direction); // Overrrides all previously set conditions
    public function addOrderBy($field, $direction);
    
    public function setMaxResults($limit);
    
    public function getQuery();
}
```

### Factories

```php
namespace Doctrine\ORM;

class EntityRepository
{
    public function createQueryBuilder($alias)
    {
        return $this->_em->createQueryBuilder()
            ->select($alias)
            ->from($this->_entityName, $alias);
    }
}
```

```php
namespace Doctrine\ORM;

class EntityManager
{
    public function createQueryBuilder()
    {
        return new QueryBuilder($this);
    }
}
```

### Code example

```php
$queryBuilder->select('count(alias.field)');
$queryBuilder->where('alias.field IN (:arrayParameter) OR alias.field LIKE :expressionParameter');
$queryBuilder->setParameters(array(
    'arrayParameter' => array(),
    'expressionParameter' => '%expr%',
));
$queryBuilder->getQuery();
```

## EntityRepository

```php
<?php

namespace Doctrine\ORM;

use Doctrine\DBAL\LockMode;
use Doctrine\Common\Collections\Selectable;
use Doctrine\Common\Collections\Criteria;
use Doctrine\Common\Persistence\ObjectRepository;

class EntityRepository implements ObjectRepository, Selectable
{
    public function getEntityManager();
    public function createQueryBuilder($alias);

    public function find($id, $lockMode = LockMode::NONE, $lockVersion = null);
    public function findAll();

    public function findBy(array $criteria, array $orderBy = null, $limit = null, $offset = null);
    public function findOneBy(array $criteria, array $orderBy = null);
    public function matching(Criteria $criteria);
}
```

### Counting

```php
<?php

use Doctrine\ORM\EntityRepository;

class MyRepository extends EntityRepository
{
    public function count()
    {
        return $this->createQueryBuilder('alias')
            ->select('count(alias.id)')
            ->getQuery()
            ->getSingleScalarResult()
        ;
    }
}
```

## Resources

* [Mastering performance](http://labs.octivi.com/mastering-symfony2-performance-doctrine/)

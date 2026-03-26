# SQL vs Prejudices

> **Talk Abstract**
>
> Silence in the court! The Supreme Court of SQL is now in session.
>
> You think you know everything about SQL, don't you? Yet SQL remains a mystery
> to most developers, accused of every wrongdoing: too complex, not modern
> enough, needlessly verbose.
>
> In this extraordinary trial, I will defend this unjustly misunderstood
> standard as a PostgreSQL expert. I will present irrefutable evidence of its
> innocence against the most common charges.
> They say SQL is obsolete: pure defamation! They claim its advanced features
> are too complex to be useful: slander! They assert that SQL is not suited to
> modern data: fake news!
>
> Through code demonstrations, real-world use cases and concrete examples, I
> will reveal how these features can transform the way you work with data.
> No endless catalogue, but practical solutions to problems you face every day.
>
> At the end of this session, you will be able to render your verdict: guilty
> of unnecessary complexity, or an elegant solution to your daily technical
> challenges?
>
> You may condemn SQL, but history will acquit it.

_Originally in French_
Speaker: Lætitia Avrot
* [Video](https://www.youtube.com/watch?v=fvD5ipYU5qE)

## Recap

Lætitia Avrot is a PostgreSQL expert, former member of the PostgreSQL Europe
board, and a recognized contributor to the PostgreSQL project.

She structures the talk as a courtroom trial, casting herself as defense
counsel for the accused: the SQL language, born in the 1970s, ISO-standardized
since 1986, present in every company she has ever worked with.

The charges: unjustified complexity, excessive verbosity, obsolescence, and
poor performance.

### The charges

The complexity charge comes from developers who find SQL concepts obscure and
would rather write a PHP `foreach` than spend three hours understanding
`PARTITION BY`.

The verbosity charge comes from queries that span two screens while equivalent
PHP fits in ten lines.

The obsolescence charge holds that ORMs and NoSQL have won, that SQL is
the COBOL of data management (Lætitia notes that given COBOL developer salaries,
this might be a compliment).

The performance charge is illustrated by a witness who put Redis in front of
PostgreSQL and concluded that SQL was the bottleneck, not the solution.

### Exhibit 1: the 50,000-update loop

The first piece of evidence is a PHP loop that iterates over 50,000 products
and issues an individual `UPDATE` for each one to apply a 5% price increase.
The problem is visible: 50,000 round trips to the database for something a
single SQL statement can do in one.

Depending on data volume, the SQL approach is 10 to 10,000 times faster.
The additional benefit is that ACID guarantees come for free.
A single `UPDATE` is atomic: either all 50,000 rows are updated, or none are.
There is no partial failure, no half-applied inflation.

### Exhibit 2: CTEs

The second exhibit is a nested subquery written in the typical developer style,
where the outermost `SELECT` appears first and the reader must work inward to
understand what is being filtered.
This is unnatural: humans read top to bottom, not inside out.

Common Table Expressions (WITH clauses) rewrite the same query in reading order:

```sql
WITH large_orders AS (
    SELECT order_id, customer_id, total
    FROM orders
    WHERE item_count >= 5
),
expensive_orders AS (
    SELECT order_id, customer_id, total
    FROM large_orders
    WHERE total > 1000
)
SELECT c.name, c.email
FROM customers c
JOIN expensive_orders eo ON c.id = eo.customer_id;
```

Each CTE is a named, scoped subquery.

The reader does not need to know what is inside a CTE to understand how it is
used.
A CTE can be referenced multiple times and the optimizer typically evaluates it
once.
For debugging, a constant table can be substituted inside the parentheses
without touching anything else.

CTEs have been part of the SQL standard since 1999.

### Exhibit 3: RETURNING

A common PHP pattern for creating a record and immediately using its ID involves
three database round trips: an `INSERT`, a call to retrieve the last inserted ID
(which is not concurrency-safe), and a `SELECT` to fetch the full record.

The `RETURNING` clause collapses this into one query:

```sql
INSERT INTO orders (customer_id, status, created_at)
VALUES (42, 'pending', NOW())
RETURNING id, order_number, created_at, status;
```

The result is guaranteed to be the row that was just inserted.
`RETURNING` works for `UPDATE` and `DELETE` as well.
PostgreSQL 18 adds `RETURNING OLD` to retrieve pre-modification values.

CTEs and `RETURNING` can be chained.
Archiving discontinued out-of-stock products in one atomic operation:

```sql
WITH to_archive AS (
    DELETE FROM products
    WHERE discontinued = true AND stock = 0
    RETURNING *
)
INSERT INTO archived_products
SELECT *, NOW() AS archived_at FROM to_archive
RETURNING id, archived_at;
```

One query, one transaction: the `DELETE`, the `INSERT` into the archive table,
and the confirmation log, with no possibility of data loss between steps.

`RETURNING` is not in the ISO SQL standard but has been adopted by PostgreSQL,
Oracle, and SQL Server. MySQL does not support it. MariaDB added partial support
from version 10.5 but not for `UPDATE`.

### Exhibit 4: window functions

The PHP approach for finding the top 3 best-selling products per category is to
fetch all products, sort them by category, and loop to extract the top 3 from
each group.
Lætitia's question: what happens when you have 100 million products and they
do not fit in memory?

Databases are built for exactly this kind of operation.

```sql
WITH ranked_products AS (
    SELECT
        product_id,
        name,
        category_id,
        total_sales,
        ROW_NUMBER() OVER (
            PARTITION BY category_id
            ORDER BY total_sales DESC, product_id
        ) AS rank
    FROM products
)
SELECT product_id, name, category_id, total_sales
FROM ranked_products
WHERE rank <= 3;
```

`ROW_NUMBER() OVER (PARTITION BY ...)` restarts the counter for each category.
The `ORDER BY` inside the `OVER` clause is required: SQL is set-based, and
without an explicit order the row sequence is undefined.
`RANK()` handles ties differently, assigning the same rank to equal values and
skipping the next one.

The `LAG()` function accesses the previous row within a partition, enabling
month-over-month comparisons without self-joins:

```sql
SELECT
    month,
    sales,
    LAG(sales) OVER (ORDER BY month) AS previous_month_sales,
    sales - LAG(sales) OVER (ORDER BY month) AS difference,
    SUM(sales) OVER (ORDER BY month) AS running_total
FROM monthly_sales;
```

A salary gap analysis using `AVG() FILTER`:

```sql
SELECT
    name,
    gender,
    job_title,
    salary,
    salary - AVG(salary) FILTER (WHERE gender = 'male')
        OVER (PARTITION BY job_title) AS gap_vs_male_average
FROM employees;
```

Window functions have been part of the SQL standard since 2003.

### Exhibit 5: LATERAL joins

The N+1 problem is one of the most common performance issues in applications
using ORMs.
For each customer, the ORM issues a separate query to fetch that customer's top
3 orders: one query to list customers, then N queries for N customers.
The ORM does this silently.
Lætitia's advice: when there is a performance problem and an ORM is involved,
blame the ORM first.

A `LATERAL` join solves this in one query.
It is a join where the right side can reference columns from the left side,
effectively running a correlated subquery for each row:

```sql
SELECT c.id, c.name, top_orders.*
FROM customers c
LEFT JOIN LATERAL (
    SELECT id, total, created_at
    FROM orders
    WHERE customer_id = c.id
    ORDER BY total DESC
    LIMIT 3
) top_orders ON true;
```

The `ON true` syntax means the join condition is always satisfied; the
filtering is handled inside the subquery.
Lætitia notes that `WHERE 1 = 1` should be written as `WHERE true` in proper
boolean logic, and the same applies here.

A more sophisticated example combines the `MODE()` aggregate (which returns the
most frequent value in a set) with a `LATERAL` join to recommend products:

```sql
WITH customer_preferences AS (
    SELECT
        c.id,
        c.name,
        MODE() WITHIN GROUP (ORDER BY oi.category_id) AS preferred_category
    FROM customers c
    JOIN order_items oi ON oi.customer_id = c.id
    GROUP BY c.id, c.name
)
SELECT cp.name, recommendations.*
FROM customer_preferences cp
LEFT JOIN LATERAL (
    SELECT p.id, p.name, SUM(oi.quantity) AS total_sold
    FROM products p
    JOIN order_items oi ON oi.product_id = p.id
    WHERE p.category_id = cp.preferred_category
    GROUP BY p.id, p.name
    ORDER BY total_sold DESC
    LIMIT 5
) recommendations ON true;
```

For each customer, this finds their most frequently purchased category and
returns the five best-selling products in that category.
The query could be refined further to exclude products the customer has already
bought.

### The real difference: imperative vs declarative

The underlying reason SQL is resisted is that it requires a paradigm shift.

PHP says: do this, then this, then this.
Every step is controlled and can be inspected.
SQL says: here is what I want, figure out how to get it.
The developer must trust the query planner to find the best path.

Lætitia describes developers as naturally inclined to be control freaks.
Letting go of that control is uncomfortable.
But in practice, the query planner is written in C by people who have spent
years optimizing it.
It is almost certainly better at finding execution paths than a handwritten
loop.
When the plan is not optimal, `EXPLAIN ANALYZE` reveals where it went wrong,
and that covers perhaps 10 to 15 percent of queries.
She mentions a query that was rewritten the previous day: from 9 hours 40
minutes down to under 2 seconds, without adding an index.

SQL's declarative nature is not a weakness.
It is what makes SQL capable of things no imperative language can match for
set-based operations.
The combination of SQL for data and a general-purpose language for everything
else produces better results than either alone.

### SQL is not the problem

CTEs have existed since 1999.
Window functions since 2003.
Lateral joins even longer.
The problem is not the language; it is that SQL education stopped in 1992.
Most developers use SQL the way it was taught a generation ago, ignoring thirty
years of additions to the standard.

### The Q&A

A developer in the audience raised the concern about maintainability when
schemas need to change: with an ORM, renaming a column triggers a migration,
updated getters and setters, and a find-and-replace; in raw SQL everything
breaks silently.

Lætitia's answer is data modeling.

Business data, meaning the data that existed before the software did, should
not need to change often.
An enterprise does not change its core activities every morning.
When a schema does keep changing, it is usually a sign that the model was not
thought through upfront.
She recommends the Merise modeling method, developed in the late 1970s with
industry practitioners specifically to produce schemas that survive into
production without modification.

The answer is not satisfying for codebases where the damage is already done,
she acknowledges, but the root cause is upstream of the ORM.

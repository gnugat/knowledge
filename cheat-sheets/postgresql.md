# PostgreSQL cheat sheet

A highly opinionated PostgreSQL cheat sheet.

## jsonb

* query an element in an array: `'["first","second"]'::jsonb->>0`
* query an element in an object: `'{"name":"Arthur Dent"}'->>'name'`
* query an element at specified path: `'{"user":{"name": "Arthur Dent"}}'::jsonb#>>'{'user','name'}'`
* query an array of objects: `jsonb_array_elements('[{"name":"Arthur Dent"},{"name":"Trillian Astra"}]'::jsonb)`
* query an element in an array of objects: `jsonb_array_elements('[{"name":"Arthur Dent"},{"name":"Trillian Astra"}]'::jsonb)->>'name'`

[Reference](https://www.postgresql.org/docs/current/static/functions-json.html)

## Monitor running transactions

Make sure you don't have a transaction that's been running for two months or something:

```sql
SELECT 
  pid,
  usename,
  age(clock_timestamp(), xact_start) AS transaction_age,
  age(clock_timestamp(), query_start) AS query_age,
  state,
  left(query, 100) AS query
FROM pg_stat_activity
WHERE xact_start IS NOT NULL
ORDER BY xact_start ASC;
```

_Reference: [Thomas Florelli](https://github.com/thomasflorelli)_

## Autovacuum health check

Identify tables accumulating dead tuples:

```sql
SELECT
  relname,
  last_autovacuum,
  ROUND(n_dead_tup::numeric / NULLIF(n_live_tup, 0), 2) AS dead_ratio,
  n_dead_tup
FROM pg_stat_user_tables
ORDER BY dead_ratio DESC;
```

> _Note_:
>
> On UPDATE / DELETE, the old version of the row becomes "dead", yet they still:
>
> - take disk space
> - increase index size
> - slow down scans
>
> That is, until VACUUM removes them. Which will NOT happen if a transaction is still running.

_Reference: [Thomas Florelli](https://github.com/thomasflorelli)_

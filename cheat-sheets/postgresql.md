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

```
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

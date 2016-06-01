# PostgreSQL cheat sheet

A highly opinionated PostgreSQL cheat sheet.

## jsonb

* query an element in an array: `'["first","second"]'::jsonb->>0`
* query an element in an object: `'{"name":"Arthur Dent"}'->>'name'`
* query an element at specified path: `'{"user":{"name": "Arthur Dent"}}'::jsonb#>>'{'user','name'}'`
* query an array of objects: `jsonb_array_elements('[{"name":"Arthur Dent"},{"name":"Trillian Astra"}]'::jsonb)`
* query an element in an array of objects: `jsonb_array_elements('[{"name":"Arthur Dent"},{"name":"Trillian Astra"}]'::jsonb)->>'name'`

[Reference](https://www.postgresql.org/docs/current/static/functions-json.html)

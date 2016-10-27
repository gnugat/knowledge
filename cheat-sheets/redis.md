# Redis cheat sheet

A highly opinionated Redis cheat sheet.

## Commands

* `SET key value [EX seconds] [PX milliseconds] [NX|XX]`
  - creates a non existing key with given string value
  - updates an existing key with given string value
* `MSET key value [key value ...]`
  - creates non existing keys with given associated string value
  - updates existing keys with given associated string value
* `GET key`
  - returns the string value of an existing key
  - returns `nil` for a non existing key
* `MGET key [key ...]`
  - returns the string values of existing keys
  - returns `nil` for non existing keys
* `DEL key [key...]`
  - removes the specified keys
  - ignores non existing keys

# log cheat sheet

A highly opinionated log cheat sheet:

* [log levels](#log-levels)

## Log levels

* `debug`
* `info`: interresting event
* `notice`: interresting event, but occasional
* `warning`: occasional event, shouldn't impact the user
* `error`: runtime error
* `critical`: serious error, should be fixed [ASAP as possible](https://media.riffsy.com/images/e0d3983c5a8808a67b69a2b29be2c009/raw)
* `alert`: very serious error, should be fixed urgently
* `emergency`: nothing works anymore

## References

* [log hunting](https://speakerdeck.com/odolbeau/logs-hunting)
* in french - [Monitorer sa prod](https://speakerdeck.com/lyrixx/symfony-live-2015-paris-monitorer-sa-prod)
* [Using logs to build a solid data infrastructure ](http://www.confluent.io/blog/using-logs-to-build-a-solid-data-infrastructure-or-why-dual-writes-are-a-bad-idea/)

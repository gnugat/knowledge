# HTTP cheat sheet

A highly opinionated HTTP cheat sheet:

* [Request methods](#request-methods)
* [Response status codes](#response-status-codes)
  * [2xx Success](#2xx-success)
  * [3xx Redirection](#3xx-redirection)
  * [4xx Client error](#4xx-client-error)
  * [5xx Server error](#5xx-server-error)
* [Form content types](#form-content-types)
* [Vocabulary](#vocabulary)
* [CORS](#cors)
* [cache](#cache)
* [Resources](#resources)

## Request methods

* `OPTIONS`: ask options and/or requirements associated with a resource [safe] [idempotent]
* `GET`: retrieve a resource's information [safe] [idempotent]
* `HEAD`: like `GET` but with headers only [idempotent]
* `POST`: create a new resource
* `PUT`: update a resource [idempotent]
* `DELETE`: remove a resource [idempotent]

## Response status codes

### 2xx Success

* `200 OK`
* `201 CREATED`
* `202 ACCEPTED`: used for delayed actions (think promises, message queues, etc)
* `204 NO CONTENT`: usually used after deletion

### 3xx Redirection

* `301 MOVED PERMANENTLY`: when a URL have been renamed
* `302 FOUND`: temporary redirection (e.g. after form submission)

### 4xx Client error

* `400 BAD REQUEST`: request with malformed syntax (example: invalid JSON)
* `401 UNAUTHORIZED`: authentication issue (missing or invalid credentials)
* `403 FORBIDDEN`: authorization issue (credential found and valid, but not allowed for this resource)
* `404 NOT FOUND`
* `409 CONFLICT` related to the state of the resource (think Finite State Machine)
* `422 UNPROCESSABLE ENTITY`: error in domain (think validation) [RFC 2518 - WebDAV]

### 5xx Server error

* `500 INTERNAL SERVER ERROR`: exceptions, crash
* `503 SERVICE UNAVAILABLE`: maintenance

## Form content types

By default, values submitted by an HTML form will have the one of the following content type:

    Content-Type: application/x-www-form-urlencoded; charset=UTF-8
    Content-Type: multipart/form-data; charset=utf-8

Using javascript, it can be changed to:

    Content-Type: application/json; charset=utf-8

## Vocabulary

* `entity`: data which can be sent or received with an HTTP message
* `idempotent`: can be called many times without different outcomes
* `resource`: data object or service identified by an URI
* `safe`: do not modify resources

## CORS

### Response

#### Access-Control-Expose-Headers

By default the following 6 response headers are allowed to be exposed:

* `Cache-Control`
* `Content-Language`
* `Content-Type`
* `Expires`
* `Last-Modified`
* `Pragma`

Any other headers need to listed in `Access-Control-Expose-Headers`. For example:

```
HTTP/1.1 204 NO CONTENT
Vary: X-Api-Key
Content-Length: 0
Access-Control-Expose-Headers: Vary, Content-Length
```

## Cache

> **TL;DR**: To cache HTTP response, set `Cache-Control` or `Etag` HTTP header in Response
> and put CloudFront/nginx/varnish in front of the server.

The HTTP protocol also specifies how to cache responses:

![sequence diagram](http://www.websequencediagrams.com/cgi-bin/cdraw?lz=SFRUUCBHYXRld2F5IENhY2hlCgpDbGllbnQtPgAKBTogUmVxdWVzdAoAGQUtPlNlcnZlcjogRm9yd2FyZGVkIHIAGgcAFAYALQtzcG9uc2UAMQgAUQYAKw4AGggAZA5TYW1lAE0JAC8PAIEfBQAwCw&s=napkin)

The Client is usually a browser and the Cache is usually a reverse proxy (like [nginx](http://nginx.org/en/),
[Varnish](https://www.varnish-cache.org/), a CDN like [cloudfront](https://aws.amazon.com/cloudfront/) or
`AppCache` embed in [Symfony](http://symfony.com/doc/current/book/http_cache.html#symfony-reverse-proxy)).

Its advantages:

* common knowledge (highly documented on the web)
* reduces the response time (avoids calling the dynamic logic)

Its drawbacks:

* requires an extra application (the Cache application, also called Proxy)
* does not reduce network calls (you still need to send a request to a proxy and wait for the cached response to be returned)
* needs more thinking (define a response staleness and freshness)

### Strategies

It provides the possibility to choose different strategies:

* Expiration (the given response should be cached for X seconds)
* Validation (is there a new version of the response?)

#### Expiration

If a response can be safely cached for a fixed period of time (e.g. 10 minutes), use `Cache-Control` HTTP Header:

```
HTTP/1.1 200 OK
Cache-Control: max-age=600
Content-Type: application/json
 
{"id":42,"name":"Arthur Dent"}
```

> **Note**: the Cache application will add a header to the response indicating its age:
>
> ```
> HTTP/1.1 200 OK
> Age: 23
> Cache-Control: max-age=600
> Content-Type: application/json
>  
> {"id":42,"name":"Arthur Dent"}
> ```

If a response can be safely cached until a known date (e.g. the 30th of October 1998, at 2:19pm),
use `Expires` HTTP Header:

```
HTTP/1.1 200 OK
Expires: Fri, 30 Oct 1998 14:19:41 GMT
Content-Type: application/json
 
{"id":42,"name":"Arthur Dent"}
```

#### Validation

If a response can be cached until told otherwise, use `Last-Modified` (a date) or/and `ETag` (a hash) HTTP Headers:

```
HTTP/1.1 200 OK
Last-Modified: Tue, 8 Sep 2015 13:35:23 GMT
ETag: a3e455afd
Content-Type: application/json
 
{"id":42,"name":"Arthur Dent"}
```

> **Note**: the Cache application will add a header to requests with `If-Modified-Since` (a date)
> or/and `If-None-Match` (a hash):
>
> ```
> GET /v1/users/42 HTTP/1.1
> If-Modified-Since: Tue, 8 Sep 2015 13:35:23 GMT
> If-None-Match: a3e455afd
> Accept: application/json
> ```
>
> If the response shouldn't be updated, the Server should return a `304 Not Modified` response,
> the Cache will then return the cached response.

### Cache-Control values

Keep in mind that `Cache-Control` header can be used by both the Cache and the Client (e.g. a browser)
to cache the response.

* `max-age=[seconds]`: How long the response should be cached
* `s-maxage=[seconds]`: Same as max-age but for Cache only
* `private`: Only the Client can cache (default for authenticated responses)
* `public`: Proxy can also Cache (for authenticated responses)
* `no-cache`: Refresh the cached copy (but still caches it)
* `no-store`: Response should not be cached anywhere (e.g. because it contains sensitive data)
* `must-revalidate`: Refresh the cached copy if it is stale
* `proxy-revalidate`: Same as must-revalidate but for Cache only

### HTTP date format

The date MUST be represented in Greenwich Mean Time (GMT), with the following format: `D, d M Y H:i:s GMT`

* `D`: A textual representation of a day, three letters (`Mon` to `Sun`)
* `d`: Day of the month, 2 digits with leading zeros (`01` to `31`)
* `M`: A short textual representation of a month, three letters (`Jan` to `Dec`)
* `Y`: A full numeric representation of a year, 4 digits (e.g. `1999`, `2003`)
* `H`: 24-hour format of an hour with leading zeros (`00` to `23`)
* `i`: Minutes with leading zeros (`00` to `59`)
* `s`: Seconds, with leading zeros (`00` to `59`)

## Resources

* [Pretty RFC 2616 - HTTP/1.1](https://pretty-rfc.herokuapp.com/RFC2616)
* [REST API Tutorial](http://www.restapitutorial.com/httpstatuscodes.html)
* [Draft HTTP/2](http://http2.github.io/)
* [HTTP 1.1 (RFC 2616), caching chapter](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13)
* [Things caches do](http://2ndscale.com/rtomayko/2008/things-caches-do)
* [Caching tutorial](https://www.mnot.net/cache_docs/)

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

* `400 BAD REQUEST`: request with malformed syntax
* `401 UNAUTHORIZED`: authenticated, but not permitted
* `403 FORBIDDEN`: not authenticated (and authentication required)
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

## Resources

* [Pretty RFC 2616 - HTTP/1.1](https://pretty-rfc.herokuapp.com/RFC2616)
* [REST API Tutorial](http://www.restapitutorial.com/httpstatuscodes.html)
* [Draft HTTP/2](http://http2.github.io/)

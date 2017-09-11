# HTTP CORS

## Response

### Access-Control-Expose-Headers

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

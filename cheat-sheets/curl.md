# curl cheat sheet

A highly opinionated curl cheat sheet.

* request header: `-H <header>`
* method: `-X <method>`
* post data: `-d <data>`
* show response headers: `-i`
* follow redirect: `-L`

Example:

    curl -d 'key1=value&key2=value' -H 'Content-Type: application/x-www-form-urlencoded; charset=UTF-8' http://example.com

Reference: http://www.cheatography.com/ankushagarwal11/cheat-sheets/curl-cheat-sheet/

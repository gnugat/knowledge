# PHP cheat sheet

A highly opinionated PHP cheat sheet.

* [array_merge](#array_merge)

## array_merge

array_merge + "splat" operator will reduce a bidimensonal array to a flattened array:

```
<?php
array_merge(... [[1, 2], [3], [4, 5]]); // [1, 2, 3, 4, 5];
```

_source: [PHP doc](https://www.php.net/manual/en/function.array-merge.php#121023)_

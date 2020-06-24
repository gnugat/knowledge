# PHP cheat sheet

A highly opinionated PHP cheat sheet.

* [array_merge](#array_merge)

## array_merge

`+` operator will merge arrays, in case of duplicate keys, preserves the element from the **left-hand** array:

```
[1, 2, 'foo' => 'bar'] + [3, 4, 5, 'foo' => 'baz'] // [1, 2, 5, 'foo' => 'bar']
```

`array_merge` will merge arrays, in case of duplicate keys, overwrites the element using the **right-hand** array:

```
[1, 2, 'foo' => 'bar'] + [3, 4, 5, 'foo' => 'baz'] // [1, 2, 3, 4, 5, 'foo' => 'baz']
```

`array_merge` + "splat" operator will reduce a bidimensonal array to a flattened array:

```
array_merge(... [[1, 2], [3], [4, 5]]); // [1, 2, 3, 4, 5];
```

_References:_

* [StackOverflow: Difference between `+` and `array_merge`](http://stackoverflow.com/a/2140094/3437428)
* [PHP: Array operators](http://php.net/manual/en/language.operators.array.php)
* [PHP: `array_merge`](http://php.net/manual/en/function.array-merge.php)
* [PHP: `array_merge` splat operator comment](https://www.php.net/manual/en/function.array-merge.php#121023)

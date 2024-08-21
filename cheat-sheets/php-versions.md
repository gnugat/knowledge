# PHP versions cheat sheet

A highly opinionated PHP versions cheat sheet:

* [PHP 8.3](#php-83)
* [PHP 8.2](#php-82)
* [PHP 8.1](#php-81)
* [PHP 8.0](#php-80)
* [PHP 7.4](#php-74)

## PHP 8.3

* [typed class constants](https://wiki.php.net/rfc/typed_class_constants) 💙
* [`#[\Override]` attribute](https://wiki.php.net/rfc/marking_overriden_methods)
* [negative array indices](https://wiki.php.net/rfc/negative_array_index)

Source: [What's new in PHP 8.3](https://stitcher.io/blog/new-in-php-83) from [Brent Roose](https://github.com/brendt)

## PHP 8.2

* [`readonly` classes](https://wiki.php.net/rfc/readonly_classes) 😡
* [`bool`, `false`, `null` typehints](https://wiki.php.net/rfc/null-false-standalone-types)
* [`#[\SensitiveParameter]` attribute](https://wiki.php.net/rfc/redact_parameters_in_back_traces)

Source: [What's new in PHP 8.2](https://stitcher.io/blog/new-in-php-82) from [Brent Roose](https://github.com/brendt)

## PHP 8.1

* [`readonly` properties](https://wiki.php.net/rfc/readonly_properties_v2)
* [`new` in initializers](https://wiki.php.net/rfc/new_in_initializers)
* [`never` return typehint](https://wiki.php.net/rfc/noreturn_type)

Source: [What's new in PHP 8.1](https://stitcher.io/blog/new-in-php-81) from [Brent Roose](https://github.com/brendt)
Also check:
* [PHP 8.1: new in initializers](https://stitcher.io/blog/php-81-new-in-initializers)

## PHP 8.0

* [union types](https://wiki.php.net/rfc/union_types_v2) 😐
* [nullsafe operator](https://wiki.php.net/rfc/nullsafe_operator)
* [named parameters](https://wiki.php.net/rfc/named_params) 💙
* [attributes](https://wiki.php.net/rfc/attributes_v2)
* [`match` expression](https://wiki.php.net/rfc/match_expression_v2)
* [constructor property promotion](https://wiki.php.net/rfc/constructor_promotion) 💙
* [`static` return type](https://wiki.php.net/rfc/static_return_type)
* [`mixed` return type](https://wiki.php.net/rfc/mixed_type_v2)
* [`::class` on object instances](https://wiki.php.net/rfc/class_name_literal_on_object)
* [trailing comma in parameter list](https://wiki.php.net/rfc/trailing_comma_in_parameter_list) 💙
* [`str_contains(string $haystack, string $needle): bool`](https://wiki.php.net/rfc/str_contains)
* [`str_starts_with(string $haystack, string $needle): bool` and `str_ends_with(string $haystack, string $needle): bool`](https://wiki.php.net/rfc/add_str_starts_with_and_ends_with_functions)
* [`ext-json` always installed](https://wiki.php.net/rfc/always_enable_json)

Source: [What's new in PHP 8](https://stitcher.io/blog/new-in-php-8) from [Brent Roose](https://github.com/brendt)
Also check:
* [PHP 8: the null safe operator](https://stitcher.io/blog/php-8-nullsafe-operator)
* [PHP 8: Attributes](https://stitcher.io/blog/attributes-in-php-8)
* [PHP 8: match or switch?](https://stitcher.io/blog/php-8-match-or-switch)

## PHP 7.4

* [arrow functions](https://wiki.php.net/rfc/arrow_functions_v2)
* [typed class properties](https://wiki.php.net/rfc/typed_properties_v2) 💙
* [null coalescing assignment operator](https://wiki.php.net/rfc/null_coalesce_equal_operator)
* [`...` array spread operator](https://wiki.php.net/rfc/spread_operator_for_array)

Source: [What's new in PHP 7.4](https://stitcher.io/blog/new-in-php-74) from [Brent Roose](https://github.com/brendt)
Also check:
* [Arrow functions in PHP 7.4](https://stitcher.io/blog/short-closures-in-php)

# increment
**```(Requires Workerman version >= 3.3.0)```**
```php
bool \GlobalData\Client::increment(string $key[, int $step = 1])
```

Atomic increment. Increases a numerical element by the size specified in the $step parameter. If the value of the element is not a numeric type, it is treated as 0 before the increment operation. Returns false if the element does not exist.

## Parameters
``` $key ```

The key. (For example, in `$global->abc`, `abc` is the key)

``` $value ```

The size by which to increment the element's value.

## Return Value
Returns true on success, false otherwise.

## Example
```php
$global = new GlobalData\Client('127.0.0.1:2207');

$global->some_key = 0;

// Non-atomic increment
$global->some_key++;

echo $global->some_key."\n";

// Atomic increment
$global->increment('some_key');

echo $global->some_key."\n";
```

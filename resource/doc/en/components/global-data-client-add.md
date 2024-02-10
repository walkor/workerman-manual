# add
**``` (Requires Workerman version >= 3.3.0) ```**
```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
Atomic addition. Returns false if the key already exists.

## Parameters

``` $key ```

The key. (For example, in `$global->abc`, `abc` is the key.)

``` $value ```

The value to be stored.

## Return Value
Returns true on success, false otherwise.

## Example
```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // Successfully assigned $global->some_key
    echo "add success " , $global->some_key;
}
else
{
    // $global->some_key already exists, assignment failed
    echo "add fail " , var_export($global->some_key);
}
```

# onError
## Description:
```php
callback Worker::$onError
```

Emitted when an error occurs.


## Parameters

``` $connection ```

The instance of Connection.

``` $code ```

Error code

``` $msg ```

Error message


## Examples

```php
use Workerman\Worker;
$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function($connection, $code, $msg)
{
    echo "error $code $msg\n";
};
```

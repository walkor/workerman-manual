# onClose
## Description:
```php
callback Connection::$onClose
```

Is the same as ```$worker->onClose```, but only for the current connection.

## Examples

```php
use Workerman\Worker;
$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnection = function($connection)
{
    $connection->onClose = function($connection)
    {
        echo "connection closed\n";
    };
};
```

Is the same as

```php
use Workerman\Worker;
$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function($connection)
{
    echo "connection closed\n";
};
```

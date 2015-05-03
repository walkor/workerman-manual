# protocol

## Description:
```php
string Connection::$protocol
```

The protocol of the connection.


## Examples


```php
use Workerman\Worker;
$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    $connection->protocol = 'Workerman\\Protocols\\Http';
};
$worker->onMessage = function($connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};
```

# Protocol

## Description:
```php
string Connection::$protocol
```

Set the protocol class for the current connection


## Example


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    $connection->protocol = 'Workerman\\Protocols\\Http';
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    // When sending, $connection->protocol::encode() will be called automatically to pack the data before sending
    $connection->send("hello");
};
// Run the worker
Worker::runAll();
```

# maxPackageSize

## Description:
```php
static int Connection::$defaultMaxPackageSize
```

This property is a global static property used to set the maximum package length that each connection can receive. If not set, the default value is 10MB.

If the parsed packet length (the return value of the protocol's input method) is greater than ```Connection::$defaultMaxPackageSize```, it will be treated as illegal data and the connection will be disconnected.


## Example


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Set the maximum data package for each connection to 1024000 bytes
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// Run the worker
Worker::runAll();
```

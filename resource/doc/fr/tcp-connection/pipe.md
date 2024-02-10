# pipe
## Description:
```php
void Connection::pipe(TcpConnection $target_connection)
```



## Arguments
Imports the data stream of the current connection into the target connection. Built-in flow control. This method is very useful for TCP proxying.



## Example: TCP Proxy

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// After the TCP connection is established
$worker->onConnect = function(TcpConnection $connection)
{
    // Establish an asynchronous connection to the local port 80
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // Set to import the data of the current client connection to the 80 port connection
    $connection->pipe($connection_to_80);
    // Set to import the data returned by the 80 port connection to the client connection
    $connection_to_80->pipe($connection);
    // Perform asynchronous connection
    $connection_to_80->connect();
};

// Run the worker
Worker::runAll();
```

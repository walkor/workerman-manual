# Pipe

## Description
```php
void Connection::pipe(TcpConnection $target_connection)
```

## Parameters
Imports the data stream of the current connection into the target connection. Built-in traffic control. This method is very useful for TCP proxy.

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
    // Set the data from the current client connection to be directed to the connection on port 80
    $connection->pipe($connection_to_80);
    // Set the data returned from the 80 port connection to be directed to the client connection
    $connection_to_80->pipe($connection);
    // Execute the asynchronous connection
    $connection_to_80->connect();
};

// Run the worker
Worker::runAll();
```

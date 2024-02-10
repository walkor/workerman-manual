# Object and Resource Persistence

In traditional web development, the objects, data, and resources created by PHP are released after each request, making it difficult to achieve persistence. However, in Workerman, it is easy to achieve persistence.

In Workerman, if you want to permanently store certain data resources in memory, you can place the resources in global variables or in static class members.

For example, in the following code:

Use a global variable ```$connection_count``` to store the current number of client connections in the process.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Global variable to store the current number of client connections in the process
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // When a new client connection is established, increment the connection count by 1
    global $connection_count;
    ++$connection_count;
    echo "now connection_count=$connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // When a client is closed, decrement the connection count by 1
    global $connection_count;
    $connection_count--;
    echo "now connection_count=$connection_count\n";
};

```

## PHP Variable Scope
See: https://php.net/manual/zh/language.variables.scope.php

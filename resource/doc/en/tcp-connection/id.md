# id

## Description:
```php
int Connection::$id
```

The id of the connection, which is an auto-incrementing integer.

Note: Workerman is a multi-process framework, and each process maintains its own auto-incrementing connection id. Therefore, there may be duplicate connection ids across multiple processes. To ensure unique connection ids, you can reassign `connection->id` as needed, such as by adding the `worker->id` prefix.

## See also
[Worker's connections property](../worker/connections.md)

## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// Run the worker
Worker::runAll();
```

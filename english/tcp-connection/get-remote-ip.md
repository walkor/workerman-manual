# getRemoteIp
## Description
```php
string Connection::getRemoteIp()
```
Get the client IP of this connection.

## Parameters
No parameters

## Example
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "new connection from ip " . $connection->getRemoteIp() . "\n";
};
// Run worker
Worker::runAll();
```

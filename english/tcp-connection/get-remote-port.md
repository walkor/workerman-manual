# getRemotePort
## Description:
```php
int Connection::getRemotePort()
```
Get the remote port of the connection.

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
    echo "new connection from address " .
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
// Run the worker
Worker::runAll();
```

# destroy
## Description:
```php
void Connection::destroy()
```
Immediately close the connection.

The difference between destroy and close is that after calling destroy, even if there is data in the send buffer of the connection that has not been sent to the peer, the connection will be closed immediately, and the ```onClose``` callback of the connection will be triggered immediately.

## Parameter

No parameters

## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // if something wrong
    $connection->destroy();
};
// Run the worker
Worker::runAll();
```

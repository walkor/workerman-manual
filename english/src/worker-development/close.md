# close
## Description:
```php
void Connection::close(mixed $data = '')
```

Sends a FIN packet.

```$connection->onClose``` will be called when send buffer is mepty.

## Parameters

``` $data ```

Optional data.

If the protocol is setted, ```$data``` will be encoded with ```Protocol::encode($data)``` before send.

## Return Values


## Examples

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function($connection, $data)
{
    // hello\n Will be encode by \Workerman\Protocols\Websocket::encode before to be sent
    $connection->close("hello\n");
};

// Run all workers
Worker::runAll();
```

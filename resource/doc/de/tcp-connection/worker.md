# Worker
## Erklärung:
```php
Worker Connection::$worker
```

Dies ist eine schreibgeschützte Eigenschaft, die die worker-Instanz angibt, zu der die aktuelle Verbindungsinstanz gehört.

## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// Wenn ein Client Daten sendet, werden sie an alle anderen vom aktuellen Prozess verwalteten Clients weitergeleitet
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// Worker ausführen
Worker::runAll();
```

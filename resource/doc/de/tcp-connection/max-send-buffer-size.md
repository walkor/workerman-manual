# maxSendBufferSize
## Erklärung:
```php
int Connection::$maxSendBufferSize
```

Jede Verbindung verfügt über einen separaten Anwendungsschicht-Sende-Buffer. Wenn die Client-Empfangsgeschwindigkeit geringer ist als die Server-Sende­geschwindigkeit, werden die Daten im Anwendungsschicht-Buffer zwischengespeichert, bis sie gesendet werden.

Dieses Attribut wird verwendet, um die Größe des Anwendungsschicht-Sende-Buffer für die aktuelle Verbindung festzulegen. Wenn dies nicht festgelegt ist, beträgt der Standardwert [Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md) (1 MB).

Dieses Attribut beeinflusst das [onBufferFull](../worker/on-buffer-full.md) Callback.

## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Setzen der Größe des Anwendungsschicht-Sende-Buffer für die aktuelle Verbindung auf 102400 Bytes
    $connection->maxSendBufferSize = 102400;
};
// Worker starten
Worker::runAll();
```

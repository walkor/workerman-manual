# defaultMaxSendBufferSize
## Beschreibung:
```php
static int Connection::$defaultMaxSendBufferSize
```

Dies ist ein globales statisches Attribut, das zur Festlegung der standardmäßigen Größe des Anwendungs-Echo-Sendepuffers für alle Verbindungen verwendet wird. Wenn nicht eingestellt, beträgt der Standardwert ```1MB```. ```Connection::$defaultMaxSendBufferSize``` kann dynamisch festgelegt werden und gilt nur für neu entstandene Verbindungen.

Dieses Attribut beeinflusst den [onBufferFull](../worker/on-buffer-full.md) Callback.

## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Setzen der standardmäßigen Größe des Anwendungs-Echo-Sendepuffers für alle Verbindungen
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Setzen der Größe des Anwendungs-Echo-Sendepuffers für die aktuelle Verbindung, überschreibt den Standardwert
    $connection->maxSendBufferSize = 4*1024*1024;
};
// Worker ausführen
Worker::runAll();
```

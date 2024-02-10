# onClose
## Erklärung:
```php
Callback-Verbindung::$onClose
```

Dieser Callback hat die gleiche Funktion wie der [Worker::$onClose](../worker/on-close.md) Callback, mit dem Unterschied, dass er nur für die aktuelle Verbindung gültig ist, das heißt, er kann für eine bestimmte Verbindung eingestellt werden.

## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Wenn es eine Verbindungsereignis gibt, wird dies ausgelöst
$worker->onConnect = function(TcpConnection $connection)
{
    // Setzen Sie den onClose-Callback für die Verbindung
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "Verbindung geschlossen\n";
    };
};
// Worker ausführen
Worker::runAll();
```

Der obige Code hat die gleiche Wirkung wie der folgende Code

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Setze den onClose-Callback für alle Verbindungen
$worker->onClose = function(TcpConnection $connection)
{
    echo "Verbindung geschlossen\n";
};
// Worker ausführen
Worker::runAll();
```

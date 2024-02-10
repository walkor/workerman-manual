# getRemoteIp
## Beschreibung:
```php
string Connection::getRemoteIp()
```

Ermittelt die IP-Adresse des Clients, der mit dieser Verbindung verbunden ist.

## Parameter

Keine Parameter

## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "Neue Verbindung von IP " . $connection->getRemoteIp() . "\n";
};
// Worker ausführen
Worker::runAll();
```

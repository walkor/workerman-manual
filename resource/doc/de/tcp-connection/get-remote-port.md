# getRemotePort
## Beschreibung:
```php
int Connection::getRemotePort()
```

Holen Sie sich den Remote-Port dieser Verbindung

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
    echo "Neue Verbindung von der Adresse " .
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
// Führen Sie den Worker aus
Worker::runAll();
```

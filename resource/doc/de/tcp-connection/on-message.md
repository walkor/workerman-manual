# onMessage
## Beschreibung:
```php
Rückruf Connection::$onMessage
```

Hat die gleiche Funktion wie das [Worker::$onMessage](../worker/on-message.md) Rückruf, der Unterschied besteht darin, dass er nur für die aktuelle Verbindung gilt, d.h. er kann für eine bestimmte Verbindung konfiguriert werden, um den onMessage Rückruf festzulegen.

## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Wenn ein Client-Verbindungsevent auftritt
$worker->onConnect = function(TcpConnection $connection)
{
    // Setzt den onMessage Rückruf für die Verbindung
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('Empfang erfolgreich');
    };
};
// Worker ausführen
Worker::runAll();
```

Der obige Code ist dasselbe wie das folgende

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Setzt den onMessage Rückruf für alle Verbindungen direkt
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('Empfang erfolgreich');
};
// Worker ausführen
Worker::runAll();
```

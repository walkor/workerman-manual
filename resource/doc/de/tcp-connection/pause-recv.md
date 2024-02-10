# pauseRecv
## Beschreibung:
```php
void Connection::pauseRecv(void)
```

Stoppt den Empfang von Daten für die aktuelle Verbindung. Das onMessage-Callback dieser Verbindung wird nicht mehr ausgelöst. Diese Methode ist sehr nützlich für die Steuerung des Upload-Traffics.

## Parameter

Keine Parameter

## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // Fügt dem connection-Objekt dynamisch ein Attribut hinzu, um zu speichern, wie viele Anfragen von dieser Verbindung empfangen wurden
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Die Verbindung empfängt nach 100 Anfragen keine weiteren Daten
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
    }
};
// Startet den Worker
Worker::runAll();
```

## Siehe auch
void Connection::resumeRecv(void) – Ermöglicht es dem entsprechenden Verbindungsobjekt, Daten wieder zu empfangen

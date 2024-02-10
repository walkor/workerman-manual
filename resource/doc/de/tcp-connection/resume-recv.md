# resumeRecv
## Erklärung:
```php
void Connection::resumeRecv(void)
```

Setzt die Verbindung fort, um Daten zu empfangen. Diese Methode wird in Verbindung mit Connection::pauseRecv verwendet und ist sehr nützlich für die Upload-Datenflusssteuerung.

## Parameter

Keine Parameter

## Beispiel

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Füge dem connection-Objekt dynamisch ein Attribut hinzu, um zu speichern, wie viele Anfragen von der aktuellen Verbindung gesendet wurden
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Die Verbindung empfängt nach 100 Anfragen keine Daten mehr
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // Datenempfang nach 30 Sekunden fortsetzen
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// Führe den Worker aus
Worker::runAll();
```

## Siehe auch
void Connection::pauseRecv(void) - Stoppt das Empfangen von Daten für das entsprechende Verbindungsobjekt

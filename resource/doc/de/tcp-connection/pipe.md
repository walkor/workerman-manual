# pipe
## Erklärung:
```php
void Connection::pipe(TcpConnection $target_connection)
```

## Parameter
Leitet den Datenstrom der aktuellen Verbindung an die Zielverbindung weiter. Die integrierte Datenverkehrssteuerung ist aktiviert. Diese Methode ist besonders nützlich für TCP-Proxy.

## Beispiel TCP-Proxy
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// Nach dem Aufbau der TCP-Verbindung
$worker->onConnect = function(TcpConnection $connection)
{
    // Aufbau einer asynchronen Verbindung zum lokalen Port 80
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // Leitet die Daten der aktuellen Client-Verbindung an die Verbindung des 80er-Ports weiter
    $connection->pipe($connection_to_80);
    // Leitet die Daten der Verbindung des 80er-Ports an die Client-Verbindung zurück
    $connection_to_80->pipe($connection);
    // Führt die asynchrone Verbindung aus
    $connection_to_80->connect();
};

// Startet den Worker
Worker::runAll();
```

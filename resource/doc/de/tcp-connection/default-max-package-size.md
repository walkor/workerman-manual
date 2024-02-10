# maxPackageSize

## Erklärung:
```php
static int Connection::$defaultMaxPackageSize
```

Dieses Attribut ist ein globales statisches Attribut, das verwendet wird, um die maximale Paketgröße festzulegen, die eine Verbindung empfangen kann. Wenn nicht festgelegt, beträgt der Standardwert 10 MB.

Wenn die Länge des empfangenen Datenpakets, das durch das Protokoll input-Methode der Klasse verarbeitet wird, größer ist als ```Connection::$defaultMaxPackageSize```, wird dies als ungültige Daten betrachtet und die Verbindung wird getrennt.

## Beispiel
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Legt die maximale Größe für empfangene Datenpakete auf 1024000 Bytes fest
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// Startet den Worker
Worker::runAll();
```

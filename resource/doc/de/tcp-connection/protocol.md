# Protokoll

## Erläuterung:
```php
string Connection::$protocol
```

Legt die Protokollklasse für die aktuelle Verbindung fest.


## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    $connection->protocol = 'Workerman\\Protocols\\Http';
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    // Beim Senden wird automatisch $connection->protocol::encode() aufgerufen, um die Daten zu verpacken und dann zu senden
    $connection->send("Hallo");
};
// Worker ausführen
Worker::runAll();
```

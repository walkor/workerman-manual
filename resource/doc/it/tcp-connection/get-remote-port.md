# getRemotePort
## Descrizione:
```php
int Connection::getRemotePort()
```

Ottiene la porta del client per questa connessione.

## Parametri

Nessun parametro

## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "nuova connessione dall'indirizzo " .
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
// Avvia il worker
Worker::runAll();
```

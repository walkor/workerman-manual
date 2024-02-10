# getRemoteIp
## Descrizione:
```php
string Connection::getRemoteIp()
```

Ottenere l'indirizzo IP del client di questa connessione

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
    echo "nuova connessione dall'ip " . $connection->getRemoteIp() . "\n";
};
// Avviare il worker
Worker::runAll();
```

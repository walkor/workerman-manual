# distruggere
## Descrizione:
```php
void Connection::destroy()
```

Chiude immediatamente la connessione.

A differenza di close, dopo aver chiamato destroy, anche se il buffer di invio della connessione contiene dati non ancora inviati al peer, la connessione sarà chiusa immediatamente e scatterà immediatamente il callback ```onClose``` di quella connessione.

## Parametri

Nessun parametro

## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // if something wrong
    $connection->destroy();
};
// Avvia il worker
Worker::runAll();
```

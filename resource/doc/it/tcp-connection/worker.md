# worker
## Descrizione:
```php
Worker Connection::$worker
```

Questa è una proprietà di sola lettura, ossia l'istanza del worker a cui l'oggetto di connessione corrente appartiene.


## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// Quando un client invia dei dati, vengono inoltrati a tutti gli altri client mantenuti dal processo corrente
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// Avvia il worker
Worker::runAll();
```

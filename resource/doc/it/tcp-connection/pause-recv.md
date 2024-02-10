# pauseRecv
## Descrizione:
```php
void Connection::pauseRecv(void)
```

Arresta la ricezione dei dati per la connessione corrente. Il callback onMessage di questa connessione non verrà attivato. Questo metodo è molto utile per il controllo del traffico in upload.

## Parametri

Nessun parametro

## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // Aggiunge dinamicamente una proprietà all'oggetto connection per memorizzare quante richieste sono state ricevute durante la connessione corrente
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Dopo aver ricevuto 100 richieste, la connessione smette di ricevere dati
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
    }
};
// Avvia il worker
Worker::runAll();
```

## Vedi anche
void Connection::resumeRecv(void) - Riprende la ricezione dei dati per l'oggetto connessione corrispondente

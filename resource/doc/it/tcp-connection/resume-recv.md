# resumeRecv
## Descrizione:
```php
void Connection::resumeRecv(void)
```

Continua a ricevere dati dalla connessione corrente. Questo metodo è utile per il controllo del traffico in upload quando utilizzato in combinazione con Connection::pauseRecv.

## Parametri

Nessun parametro

## Esempio

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Aggiunge dinamicamente una proprietà all'oggetto di connessione per memorizzare quante richieste sono state inviate attraverso la connessione
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Dopo aver ricevuto 100 richieste, la connessione smette di ricevere dati
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // Dopo 30 secondi riprende a ricevere dati
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// Avvia il worker
Worker::runAll();
```

## Vedi anche
void Connection::pauseRecv(void) Ferma la ricezione dei dati per l'oggetto di connessione corrispondente

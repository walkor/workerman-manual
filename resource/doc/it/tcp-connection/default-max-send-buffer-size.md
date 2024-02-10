# defaultMaxSendBufferSize
## Descrizione:
```php
static int Connection::$defaultMaxSendBufferSize
```

Questa proprietà è una proprietà statica globale utilizzata per impostare la dimensione predefinita del buffer di invio del livello dell'applicazione per tutte le connessioni. Se non viene impostato, il valore predefinito è ```1MB```. ```Connection::$defaultMaxSendBufferSize``` può essere impostato dinamicamente e avrà effetto solo sulle nuove connessioni create successivamente.

Questo parametro influenzerà il callback [onBufferFull](../worker/on-buffer-full.md).

## Esempio
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Imposta la dimensione predefinita del buffer di invio del livello dell'applicazione per tutte le connessioni
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Imposta la dimensione del buffer di invio del livello dell'applicazione per la connessione corrente, sovrascrivendo il valore predefinito
    $connection->maxSendBufferSize = 4*1024*1024;
};
// Avvia il worker
Worker::runAll();
```

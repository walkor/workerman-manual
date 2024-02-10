# maxSendBufferSize
## Descrizione
```php
int Connection::$maxSendBufferSize
```

Ogni connessione ha un buffer di invio a livello di applicazione separato. Se la velocità di ricezione del client è inferiore alla velocità di invio del server, i dati vengono temporaneamente memorizzati nel buffer di invio a livello di applicazione in attesa di essere inviati.

Questa proprietà viene utilizzata per impostare la dimensione del buffer di invio a livello di applicazione della connessione corrente. Se non viene impostata, il valore predefinito è [Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md) (1MB).

Questa proprietà influisce sul callback [onBufferFull](../worker/on-buffer-full.md).

## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Imposta la dimensione del buffer di invio a livello di applicazione della connessione corrente a 102400 byte
    $connection->maxSendBufferSize = 102400;
};
// Avvia il worker
Worker::runAll();
```

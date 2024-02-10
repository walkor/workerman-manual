# onBufferDrain
## Descrizione:
```php
callback Worker::$onBufferDrain
```

Ogni connessione ha un buffer di invio a livello di applicazione separato, la dimensione del buffer è determinata da ```TcpConnection::$maxSendBufferSize```, il valore predefinito è 1 MB, ma può essere modificato manualmente e avrà effetto su tutte le connessioni. 

Questo callback viene attivato quando tutti i dati nel buffer di invio a livello di applicazione sono stati inviati. Di solito viene utilizzato insieme a onBufferFull, ad esempio, interrompendo l'invio di dati all'altra parte in onBufferFull e riprendendo la scrittura dei dati in onBufferDrain.


## Parametro della funzione di callback

 ``` $connection ```

Oggetto di connessione, ovvero un'istanza di [TcpConnection](../tcp-connection.md), utilizzato per gestire la connessione del client, ad esempio [invio di dati](../tcp-connection/send.md), [chiusura della connessione](../tcp-connection/close.md), ecc.


## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "bufferFull and do not send again\n";
};
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "buffer drain and continue send\n";
};
// Avvia il worker
Worker::runAll();
```

Nota: Oltre all'uso di una funzione anonima come callback, è possibile utilizzare [altri metodi di callback](../faq/callback_methods.md).

## Vedi anche
onBufferFull Quando il buffer di invio a livello di applicazione della connessione è pieno, scatena l'evento.

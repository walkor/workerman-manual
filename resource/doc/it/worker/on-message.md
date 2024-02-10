# onMessage
## Descrizione:
```php
callback Worker::$onMessage
```

Viene richiamata la funzione di callback quando il client invia dati attraverso la connessione (quando Workerman riceve i dati).

## Parametri della funzione di callback

``` $connection ```

Oggetto di connessione, ovvero un'istanza di [TcpConnection](../tcp-connection.md), utilizzato per gestire la connessione del client, come [invio di dati](../tcp-connection/send.md), [chiusura della connessione](../tcp-connection/close.md), ecc.

``` $data ```

Dati inviati dalla connessione del client. Se il Worker ha specificato un protocollo, allora $data è il dato decodificato secondo il protocollo corrispondente. Il tipo di dati dipende dall'implementazione di `decode()` del protocollo specificato. Per i protocolli `websocket`, `test`, `frame` il dato è una stringa, mentre per il protocollo HTTP è un oggetto [`Workerman\Protocols\Http\Request`](../http/request.md).

## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('ricezione avvenuta con successo');
};
// Avvia il worker
Worker::runAll();
```

Nota: Oltre all'uso di una funzione anonima come callback, è possibile utilizzare [altre forme di callback](../faq/callback_methods.md) come indicato qui.

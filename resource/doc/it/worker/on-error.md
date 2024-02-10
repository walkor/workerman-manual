# onError
## Descrizione:
```php
callback Worker::$onError
```

Viene innescato quando si verifica un errore sulla connessione del client.

Attualmente ci sono i seguenti tipi di errore: 

1. Fallimento di Connection::send a causa della disconnessione del client (seguito immediatamente da un callback di onClose) ```(code:WORKERMAN_SEND_FAIL msg:client closed)```

2. Dopo aver innescato onBufferFull (buffer di invio pieno), viene comunque chiamato Connection::send e il buffer di invio rimane pieno, causando un fallimento nell'invio (non innescando un callback di onClose) ```(code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package)```

3. Se fallisce la connessione asincrona di AsyncTcpConnection (seguito immediatamente da un callback di onClose) ```(code:WORKERMAN_CONNECT_FAIL msg:stream_socket_client restituisce il messaggio di errore)```

## Parametri della funzione di callback
``` $connection ```

Oggetto di connessione, cioè un'istanza di [TcpConnection](../tcp-connection.md), utilizzato per operare sulla connessione del client, ad esempio [invio di dati](../tcp-connection/send.md), [chiusura della connessione](../tcp-connection/close.md), ecc.

 ``` $code ```

Codice di errore

 ``` $msg ```

Messaggio di errore

## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "errore $code $msg\n";
};
// Esegui il worker
Worker::runAll();
```

Nota: Oltre all'utilizzo di funzioni anonime come callback, è possibile utilizzare [questo metodo](../faq/callback_methods.md) per gli altri metodi di callback.

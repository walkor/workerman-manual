# onConnect
## Descrizione:
```php
callback Worker::$onConnect
```

La funzione di callback triggerata quando il client si connette a Workerman (dopo il completamento della stretta di mano TCP). Ogni connessione attiva triggererà la funzione di callback `onConnect` solo una volta.

Nota: l'evento `onConnect` rappresenta solamente il completamento della stretta di mano TCP tra il client e Workerman, in questo momento il client non ha inviato alcun dato. Pertanto, oltre al recupero dell'IP remoto tramite `$connection->getRemoteIp()`, non ci sono altri dati o informazioni per identificare il client. Per sapere chi è il mittente, è necessario che il client invii dati di autenticazione, come un token o nome utente e password, da verificare nel [callback onMessage](on-message.md).

Poiché l'UDP è senza connessione, quando viene utilizzato l'UDP, non verrà triggerata la funzione di callback `onConnect`, né quella di `onClose`.

## Parametri della funzione di callback

``` $connection ```

Oggetto di connessione, cioè l'istanza di [TcpConnection](../tcp-connection.md), utilizzata per gestire la connessione del client, ad esempio [inviare dati](../tcp-connection/send.md), [chiudere la connessione](../tcp-connection/close.md), ecc.

## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "nuova connessione dall'IP " . $connection->getRemoteIp() . "\n";
};
// Avvia il worker
Worker::runAll();
```

Nota: Oltre all'uso di una funzione anonima come callback, è possibile utilizzare [altre modalità di callback](../faq/callback_methods.md) consultando questo link.

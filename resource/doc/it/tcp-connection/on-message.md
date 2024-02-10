# onMessage
## Descrizione:
```php
callback Connection::$onMessage
```

Ha lo stesso effetto del callback [Worker::$onMessage](../worker/on-message.md), con la differenza che è valido solo per la connessione corrente, ovvero può essere impostato il callback onMessage per una specifica connessione.


## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Quando si verifica un evento di connessione del client
$worker->onConnect = function(TcpConnection $connection)
{
    // Imposta il callback onMessage per la connessione
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('ricezione riuscita');
    };
};
// Avvia il worker
Worker::runAll();
```

Il codice sopra è equivalente all'effetto seguente:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Imposta direttamente il callback onMessage per tutte le connessioni
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('ricezione riuscita');
};
// Avvia il worker
Worker::runAll();
```

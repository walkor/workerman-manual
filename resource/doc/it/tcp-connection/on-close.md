# onClose
## Descrizione:
```php
callback Connection::$onClose
```

Questo callback funziona come il callback [Worker::$onClose](../worker/on-close.md), con la differenza che è valido solo per la connessione corrente, cioè è possibile impostare il callback onClose per una specifica connessione.

## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Quando si verifica l'evento di connessione
$worker->onConnect = function(TcpConnection $connection)
{
    // Imposta il callback onClose della connessione
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "connessione chiusa\n";
    };
};
// Avvia il worker
Worker::runAll();
```

Il codice sopra produce lo stesso effetto di quanto segue

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Imposta il callback onClose per tutte le connessioni
$worker->onClose = function(TcpConnection $connection)
{
    echo "connessione chiusa\n";
};
// Avvia il worker
Worker::runAll();
```

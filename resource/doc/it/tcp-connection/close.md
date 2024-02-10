# close
## Descrizione:
```php
void Connection::close(mixed $data = '')
```

Chiude in modo sicuro la connessione.

Chiamare il metodo close farà sì che la connessione venga chiusa solo dopo che tutti i dati nel buffer di invio siano stati inviati e attiverà il callback ```onClose``` della connessione.

## Parametri

``` $data ```

Parametro opzionale, i dati da inviare (se è specificato un protocollo, verrà automaticamente chiamato il metodo encode del protocollo per impacchettare i dati di ```$data```), una volta che i dati sono stati inviati, la connessione verrà chiusa e successivamente attiverà il callback onClose.

## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// Avvia il worker
Worker::runAll();
```

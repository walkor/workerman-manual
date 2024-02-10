# connessioni
## Descrizione:
```php
array Worker::$connections
```

Il formato è
```php
array(id=>connessione, id=>connessione, ...)
```

Questa proprietà memorizza tutti gli oggetti di connessione dei client **attuali** del processo. In questo contesto, l'id è il numero di identificazione della connessione, per ulteriori dettagli consultare il manuale della proprietà [id di TcpConnection](../tcp-connection/id.md).

```$connections``` è molto utile durante la trasmissione broadcast o per ottenere l'oggetto di connessione in base all'id della connessione.

Se si conosce l'id della connessione come ```$id```, è possibile ottenere comodamente l'oggetto di connessione corrispondente tramite ```$worker->connections[$id]```, in modo da operare sulla connessione del socket corrispondente, ad esempio, inviando dati tramite ```$worker->connections[$id]->send('...')```.

Nota: Se la connessione viene chiusa (scatenando onClose), la corrispondente ```connessione``` verrà rimossa dall'array ```$connections```.

Nota: Gli sviluppatori non devono apportare modifiche a questa proprietà, in caso contrario potrebbero verificarsi situazioni impreviste.


## Esempio
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2020');
// Quando il processo viene avviato, imposta un timer per inviare dati a tutte le connessioni client
$worker->onWorkerStart = function($worker)
{
    // Timer, ogni 10 secondi
    Timer::add(10, function()use($worker)
    {
        // Itera su tutte le connessioni client del processo corrente, inviando l'ora corrente del server
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Avvia il worker
Worker::runAll();
```

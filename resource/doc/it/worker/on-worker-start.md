# onWorkerStart
## Descrizione:
```php
callback Worker::$onWorkerStart
```

Imposta la funzione di callback per l'avvio del processo figlio del Worker, che verrà eseguita ogni volta che un processo figlio viene avviato.

Nota: onWorkerStart viene eseguito quando il processo figlio viene avviato. Se sono attivi più processi figlio (`$worker->count > 1`), la funzione verrà eseguita `$worker->count` volte in totale.


## Parametri della funzione di callback

 ``` $worker ```

Rappresenta l'oggetto Worker.


## Esempio

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "Avvio del worker...\n";
};
// Avvia il worker
Worker::runAll();
```

Suggerimento: Oltre all'utilizzo di funzioni anonime come callback, è possibile [fare riferimento a questo link](../faq/callback_methods.md) per un'altra sintassi di callback.

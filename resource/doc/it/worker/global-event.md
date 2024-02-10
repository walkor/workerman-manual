# globalEvent

## Descrizione:
```php
static Event Worker::$globalEvent
```

Questo attributo è un attributo statico globale e rappresenta un'istanza globale di eventloop, che consente di registrare eventi di lettura/scrittura del descrittore di file o eventi di segnale.

## Esempio

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'Il PID è ' . posix_getpid() . "\n";
    // Quando il processo riceve il segnale SIGALRM, stampa alcune informazioni
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Ricevuto il segnale SIGALRM\n";
    });
};
// Avvia il worker
Worker::runAll();
```

## Test
Dopo aver avviato Workerman, verrà stampato il PID del processo corrente (un numero). Eseguendo il seguente comando da riga di comando
``` 
kill -SIGALRM PID_del_processo
```
Il server stamperà
``` 
Ricevuto il segnale SIGALRM
```

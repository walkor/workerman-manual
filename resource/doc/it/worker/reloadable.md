# reloadable
## Descrizione:
```php
bool Worker::$reloadable
```

Quando viene eseguito il comando `php start.php reload`, verrà inviato un segnale di reload (SIGUSR1) a tutti i processi figlio.

I processi figlio, una volta ricevuto il segnale di reload, termineranno automaticamente e quindi il processo principale avvierà automaticamente un nuovo processo, di solito utilizzato per aggiornare il codice di business.

Quando il processo $reloadable è impostato su false, la ricezione del segnale di reload attiverà solo [onWorkerReload](on-worker-reload.md) e non riavvierà il processo corrente.

Ad esempio, nel modello Gateway/Worker, il processo gateway gestisce la manutenzione delle connessioni dei client, mentre il processo worker si occupa delle richieste. Impostando la proprietà reloadable del processo gateway su false, è possibile aggiornare il codice di business senza interrompere la connessione con i client.

## Esempio

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Imposta se questo istanza deve riavviarsi dopo aver ricevuto il segnale di reload
$worker->reloadable = false;
$worker->onWorkerStart = function ($worker) {
    echo "Avvio del worker...\n";
};
// Esegui il worker
Worker::runAll();
```

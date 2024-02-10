# onWorkerReload

Richiesto (workerman >= 3.2.5)

## Descrizione:

```php
callback Worker::$onWorkerReload
```

Questa funzionalità non è comunemente utilizzata.

Imposta il callback da eseguire quando il Worker riceve il segnale di reload.

È possibile utilizzare il callback onWorkerReload per svolgere molte azioni, ad esempio ricaricare i file di configurazione del business senza dover riavviare il processo.

**Nota**:

Di default, quando un processo figlio riceve il segnale di reload, si chiude e riavvia in modo che il nuovo processo possa ricaricare il codice aziendale per completare l'aggiornamento del codice. Quindi, è normale che il processo figlio esegua immediatamente il callback onWorkerReload dopo il reload.

Se si desidera che il processo figlio esegua solo onWorkerReload dopo aver ricevuto il segnale di reload e non si desidera che esca, è possibile impostare la proprietà reloadable della corrispondente istanza Worker come false durante l'inizializzazione dell'istanza Worker.

## Parametri della funzione di callback

``` $worker ```

Ovvero l'oggetto Worker

## Esempio

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Imposta reloadable su false, il che significa che il processo figlio non eseguirà il riavvio quando riceve il segnale di reload
$worker->reloadable = false;
// Dopo il reload, informa tutti i client che il server ha eseguito il reload
$worker->onWorkerReload = function(Worker $worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// Esegui il worker
Worker::runAll();
```

Nota: Oltre all'uso di una funzione anonima come callback, è possibile utilizzare [altre forme di callback](../faq/callback_methods.md) come riferimento.

# count

## Descrizione:
```php
int Worker::$count
```

Imposta quante istanze di Worker avviare, di default è 1.

Per impostare il numero di processi, fare riferimento a [**questo documento**](../faq/processes-count.md).

Nota: questa proprietà deve essere impostata prima di eseguire ```Worker::runAll();```. Questa funzione non è supportata su sistemi Windows.

## Esempio

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Avvia 8 processi, ascoltando contemporaneamente sulla porta 8484 e fornendo servizio tramite il protocollo websocket
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Avvio del Worker...\n";
};
// Esegui il worker
Worker::runAll();
```

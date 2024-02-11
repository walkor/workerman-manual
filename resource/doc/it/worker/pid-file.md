# pidFile
## Descrizione:
```php
static string Worker::$pidFile
```

Se non vi è una necessità specifica, si consiglia di non impostare questa proprietà.

Questa è una proprietà statica globale utilizzata per impostare il percorso del file pid per il processo Workerman.

Questa impostazione è utile per il monitoraggio, ad esempio è possibile inserire il file pid di Workerman in una directory fissa, in modo che alcune applicazioni di monitoraggio possano leggere il file pid e monitorare lo stato del processo Workerman.

Se non viene impostato, Workerman genererà automaticamente un file pid in una posizione parallela alla directory Workerman (prima di workerman 3.2.3 la versione predefinita era in ```sys_get_temp_dir()```) e per evitare conflitti di pid dovuti all'avvio di più istanze di Workerman, il file pid di Workerman contiene il percorso attuale di Workerman.

Nota: questa proprietà deve essere impostata prima dell'esecuzione di ```Worker::runAll();```. Questa funzionalità non è supportata dal sistema Windows.

## Esempio

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$pidFile = '/var/run/workerman.pid';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Avvio del worker";
};
// Avvia il worker
Worker::runAll();
```

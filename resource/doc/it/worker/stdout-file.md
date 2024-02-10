# stdoutFile
## Descrizione:
```php
static string Worker::$stdoutFile
```

Questa è una proprietà statica globale. Se avvii il processo in modalità demone (con ```-d```), allora tutti gli output verso il terminale (come echo, var_dump, ecc.) saranno reindirizzati nel file specificato da stdoutFile.

Se non viene impostata e il processo viene avviato in modalità demone, tutti gli output del terminale verranno reindirizzati a `/dev/null` (cioè, verranno scartati di default).

> Nota: `/dev/null` è un file speciale in Linux, è un buco nero dove tutti i dati scritti vengono scartati. Se non vuoi scartare gli output, puoi impostare `Worker::$stdoutFile` su un percorso di file normale.

> Nota: Questa proprietà deve essere impostata prima dell'esecuzione di ```Worker::runAll();```. Questa funzionalità non è supportata su sistemi Windows.

## Esempio

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// Tutti gli output stampati vengono salvati nel file /tmp/stdout.log
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Avvio del worker\n";
};
// Avvia il worker
Worker::runAll();
```

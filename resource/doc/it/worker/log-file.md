# logFile
## Descrizione:
```php
static string Worker::$logFile
```

Utilizzato per specificare la posizione del file di registro di Workerman.

Questo file registra i log correlati a Workerman stesso, inclusi avvio, arresto, etc.

Se non è stato impostato, il nome del file predefinito è workerman.log e si trova nella directory superiore di Workerman.

**Nota:**

Questo file di registro registra solo i log correlati all'avvio e all'arresto di Workerman e non include nessun log di business.

I log di business possono essere implementati utilizzando le funzioni [file_put_contents](https://php.net/manual/zh/function.file-put-contents.php) o [error_log](https://php.net/manual/zh/function.error-log.php).

## Esempio

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Avvio del worker";
};
// Avvia il worker
Worker::runAll();
```

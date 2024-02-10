# logFile
## Erklärung:
```php
static string Worker::$logFile
```

Wird verwendet, um den Speicherort der Workerman-Protokolldatei anzugeben.

Diese Datei enthält Protokolle, die sich auf Workerman selbst beziehen, einschließlich des Starts, Stops usw.

Wenn kein Wert festgelegt ist, ist der Dateiname standardmäßig workerman.log und der Speicherort befindet sich im übergeordneten Verzeichnis von Workerman.

**Hinweis:**

Diese Protokolldatei enthält nur Protokolle im Zusammenhang mit dem Starten und Stoppen von Workerman und enthält keine Geschäftsprotokolle.

Geschäftsprotokolle können mithilfe von Funktionen wie [file_put_contents](https://php.net/manual/zh/function.file-put-contents.php) oder [error_log](https://php.net/manual/zh/function.error-log.php) implementiert werden.

## Beispiel

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// Worker ausführen
Worker::runAll();
```

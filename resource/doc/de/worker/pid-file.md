# pidFile
## Beschreibung:
```php
static string Worker::$pidFile
```

Es wird empfohlen, diese Eigenschaft nicht zu setzen, es sei denn, es besteht ein spezifischer Bedarf.

Dies ist eine globale statische Eigenschaft, die verwendet wird, um den Pfad zur pid-Datei des Workerman-Prozesses festzulegen.

Diese Einstellung ist besonders nützlich für die Überwachung. Indem die pid-Datei von Workerman in ein festes Verzeichnis verschoben wird, können einige Überwachungssoftwares die pid-Datei leichter lesen und so den Status des Workerman-Prozesses überwachen.

Wenn nichts festgelegt ist, generiert Workerman standardmäßig eine pid-Datei an einem Ort, der parallel zum Workerman-Verzeichnis liegt (beachten Sie, dass dies vor der Version 3.2.3 standardmäßig in ```sys_get_temp_dir()``` generiert wurde). Um Konflikte mit mehreren Workerman-Instanzen beim Start zu vermeiden, enthält die von Workerman generierte pid-Datei den aktuellen Pfad des Workerman.

Hinweis: Diese Eigenschaft muss vor dem Aufruf von ```Worker::runAll();``` festgelegt werden, um wirksam zu sein. Windows-Systeme unterstützen dieses Feature nicht.

## Beispiel

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$pidFile = '/var/run/workerman.pid';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// Worker ausführen
Worker::runAll();
```

# daemonisieren
## Erklärung:
```php
static bool Worker::$daemonize
```

Diese Eigenschaft ist eine globale statische Eigenschaft, die angibt, ob der Worker-Prozess als Daemon (Hintergrundprozess) läuft. Wenn der Startbefehl das ```-d```-Argument verwendet, wird diese Eigenschaft automatisch auf true gesetzt. Sie kann auch im Code manuell gesetzt werden.

Hinweis: Diese Eigenschaft muss vor dem Ausführen von ```Worker::runAll();``` gesetzt werden, um wirksam zu sein. Diese Funktion wird nicht von Windows unterstützt.

## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// Worker ausführen
Worker::runAll();
```

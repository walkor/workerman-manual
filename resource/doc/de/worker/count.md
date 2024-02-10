# count

## Beschreibung:
```php
int Worker::$count
```

Legt fest, wie viele Prozesse vom aktuellen Worker-Objekt gestartet werden sollen. Standardmäßig ist dies auf 1 eingestellt.

Für Informationen zur Einstellung der Prozessanzahl siehe [**hier**](../faq/processes-count.md).

Hinweis: Diese Eigenschaft muss vor dem Aufruf von ```Worker::runAll();``` festgelegt werden, um wirksam zu sein. Dieses Feature wird nicht von Windows-Systemen unterstützt.

## Beispiel

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Starten von 8 Prozessen, die gleichzeitig den Port 8484 überwachen und den Service über das WebSocket-Protokoll bereitstellen
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// Worker ausführen
Worker::runAll();
```

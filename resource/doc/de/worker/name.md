# Name

## Beschreibung:
```php
string Worker::$name
```

Setzt den Namen der aktuellen Worker-Instanz, um den Prozess beim Ausführen des "status" Befehls zu identifizieren. Wenn kein Name gesetzt ist, ist der Standardwert "none".

## Beispiel

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Setzt den Namen der Instanz
$worker->name = 'MyWebsocketWorker';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// Führt den Worker aus
Worker::runAll();
```

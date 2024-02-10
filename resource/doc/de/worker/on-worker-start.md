# onWorkerStart
## Erklärung:
```php
Rückruf Funktion Worker::$onWorkerStart
```

Setzt die Rückruffunktion, die beim Starten des Worker-Subprozesses ausgeführt wird. Diese Funktion wird jedes Mal ausgeführt, wenn ein Unterprozess gestartet wird.

Hinweis: onWorkerStart wird beim Starten des Unterprozesses ausgeführt. Wenn mehrere Unterprozesse gestartet werden (```$worker->count > 1```), wird die Funktion insgesamt ```$worker->count``` Mal ausgeführt.


## Parameter der Rückruffunktion

 ``` $worker ```

Das Worker-Objekt


## Beispiel


```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "Starte den Worker...\n";
};
// Starte den Worker
Worker::runAll();
```

Hinweis: Neben der Verwendung anonymer Funktionen als Rückruf können auch [andere Rückrufmethoden](../faq/callback_methods.md) verwendet werden.

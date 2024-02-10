# onWorkerReload
Erfordert ``` (workerman >= 3.2.5) ```
## Beschreibung:
```php
callback Worker::$onWorkerReload
```
Dieses Feature wird selten verwendet.

Legt fest, dass der Worker nach Erhalt des Reload-Signals eine Rückruffunktion ausführt.

Das onWorkerReload-Rückruf kann für viele Dinge genutzt werden, z.B. um die Geschäftskonfigurationsdateien neu zu laden, ohne den Prozess neu zu starten.

**Hinweis**:

Die Standardaktion für einen Kindprozess nach dem Empfangen des Reload-Signals besteht darin, den Prozess zu beenden und neu zu starten, damit neue Prozesse den Geschäftscode neu laden und die Codeaktualisierung abschließen können. Daher ist es normal, dass der Kindprozess nach Ausführung des onWorkerReload-Rückrufs sofort beendet.

Wenn Sie möchten, dass der Kindprozess nach Erhalt des Reload-Signals nur die onWorkerReload-Funktion ausführt und nicht beendet, können Sie beim Initialisieren der Worker-Instanz die reloadable-Eigenschaft der entsprechenden Worker-Instanz auf false setzen.

## Parameter der Rückruffunktion
 ``` $worker ```

Das Worker-Objekt

## Beispiel
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Setzen von reloadable auf false, d.h. der Kindprozess führt nach Erhalt des Reload-Signals keinen Neustart durch
$worker->reloadable = false;
// Teilt allen Clienten nach dem Ausführen des Reloads mit, dass der Server den Reload durchgeführt hat
$worker->onWorkerReload = function(Worker $worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// Starten des Workers
Worker::runAll();
```

Hinweis: Neben der Verwendung anonymer Funktionen als Rückruf können Sie auch [hier](../faq/callback_methods.md) andere Rückrufschreibweisen verwenden.

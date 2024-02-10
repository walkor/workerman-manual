# reloadable
## Erklärung:
```php
bool Worker::$reloadable
```

Wenn `php start.php reload` ausgeführt wird, wird ein Reload-Signal (SIGUSR1) an alle Kindprozesse gesendet.

Nach Erhalt des Reload-Signals wird das Kindprozess automatisch beendet und der Hauptprozess startet automatisch einen neuen Prozess, in der Regel für die Aktualisierung des Geschäftslogik-Codes.

Wenn `$reloadable` des Prozesses auf false gesetzt ist, wird nach Erhalt des Reload-Signals nur das [onWorkerReload](on-worker-reload.md) ausgelöst, und der aktuelle Prozess wird nicht neu gestartet.

Zum Beispiel, im Gateway/Worker-Modell ist der Gateway-Prozess für die Verwaltung der Client-Verbindungen zuständig, während der Worker-Prozess die Anfragen verarbeitet. Durch das Setzen der reloadable-Eigenschaft des Gateway-Prozesses auf false kann der Geschäftslogik-Code aktualisiert werden, ohne die Client-Verbindungen zu trennen.

## Beispiel

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Setzen der Eigenschaft, ob diese Instanz nach Erhalt des Reload-Signals neu gestartet werden soll
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Worker startet...\n";
};
// Starten des Workers
Worker::runAll();
```

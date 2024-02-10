# stdoutFile
## Erklärung:
```php
static string Worker::$stdoutFile
```

Diese Eigenschaft ist eine global statische Eigenschaft. Wenn im Daemon-Modus gestartet wird (mit ```-d```), werden alle Ausgaben auf der Konsole (wie echo var_dump usw.) in die von stdoutFile angegebene Datei umgeleitet.

Wenn diese Eigenschaft nicht gesetzt ist und im Daemon-Modus läuft, werden alle Konsolenausgaben standardmäßig in `/dev/null` umgeleitet (d.h. alle Ausgaben werden standardmäßig verworfen).

> Beachten: `/dev/null` ist eine spezielle Datei in Linux, die eigentlich ein "Schwarzes Loch" ist. Alle Daten, die in diese Datei geschrieben werden, werden verworfen. Wenn die Ausgabe nicht verworfen werden soll, kann Worker::$stdoutFile auf einen normalen Dateipfad gesetzt werden.

> Beachten: Diese Eigenschaft muss vor dem Aufruf von ```Worker::runAll();``` gesetzt werden, um wirksam zu sein. Dieses Feature wird von Windows nicht unterstützt.

## Beispiel

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// Alle gedruckten Ausgaben werden in der Datei /tmp/stdout.log gespeichert
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// Worker ausführen
Worker::runAll();
```

# globalEvent

## Beschreibung:
```php
static Event Worker::$globalEvent
```

Dieses Attribut ist eine globale statische Eigenschaft, die eine globale Instanz von Eventloop darstellt. Es ermöglicht die Registrierung von Lese- und Schreibereignissen für Dateideskriptoren oder Signalereignissen.

## Beispiel

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'Pid is ' . posix_getpid() . "\n";
    // Wenn der Prozess das SIGALRM-Signal empfängt, gibt er einige Informationen aus
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Get signal SIGALRM\n";
    });
};
// Worker ausführen
Worker::runAll();
```

## Test
Nach dem Start von Workerman wird die Prozess-ID (eine Nummer) ausgegeben. Führen Sie den Befehl in der Befehlszeile aus:
```bash
kill -SIGALRM Prozess-ID
```
Der Server gibt dann aus:
```bash
Get signal SIGALRM
```

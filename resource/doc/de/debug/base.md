# Grundlegende Fehlerbehebung

Workerman hat zwei Betriebsmodi, den Debug-Modus und den Daemon-Betriebsmodus.

Führen Sie ```php start.php start``` aus, um den Debug-Modus zu starten. In diesem Modus werden Ausgaben von Funktionen wie ```echo```, ```var_dump``` und ```var_export``` in der Terminalanzeige angezeigt. Beachten Sie, dass Workerman, der mit ```php start.php start``` gestartet wird, alle Prozesse beendet, wenn das Terminal geschlossen wird.

Laufen Sie hingegen mit ```php start.php start -d```, um den Daemon-Betriebsmodus zu starten, der der offizielle Online-Betriebsmodus ist und nicht von der Schließung des Terminals beeinflusst wird.

Wenn Sie im Daemon-Modus trotzdem Ausgaben von Funktionen wie ```echo```, ```var_dump```, und ```var_export``` sehen möchten, können Sie die Worker::$stdoutFile-Eigenschaft festlegen, beispielsweise:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Leiten Sie die Bildschirmausgabe in die von Worker::$stdoutFile angegebene Datei um
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hallo Welt');
};

Worker::runAll();
```
Auf diese Weise werden alle Ausgaben von Funktionen wie ```echo```, ```var_dump```, und ```var_export```, in die in ```Worker::$stdoutFile``` angegebene Datei geschrieben. Beachten Sie, dass der angegebene Pfad von ```Worker::$stdoutFile``` Schreibrechte haben muss.

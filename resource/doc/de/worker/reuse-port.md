# reusePort
> **Hinweis**
> Erfordert Workerman >= 3.2.1  PHP >= 7.0. Diese Funktion wird nicht von Windows-Systemen und Mac OS unterstützt.

## Beschreibung:

```php
bool Worker::$reusePort
```

Legt fest, ob der aktuelle Worker das Wiederverwenden des Portüberwachungsports (SO_REUSEPORT-Option des Sockets) aktiviert.

Durch das Aktivieren des Wiederverwendens des Portüberwachungsports können mehrere nicht verwandte Prozesse denselben Port überwachen, und das Betriebssystem-Kernel entscheidet, welcher Prozess die Socket-Verbindung behandeln wird, um den Effekt des "Thundering Herd"-Phänomens zu vermeiden und die Leistung von Multi-Prozess-Kurzverbindungsanwendungen zu verbessern.

**Hinweis:** Diese Funktion erfordert PHP-Versionen >= 7.0.

## Beispiel 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// Worker ausführen
Worker::runAll();
```

## Beispiel 2: Workerman überwacht mehrere Ports (mehrere Protokolle)

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// Nachdem jeder Prozess gestartet wurde, einen neuen Port im aktuellen Prozess hinzufügen
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * Mehrere Prozesse überwachen denselben Port (der Überwachungssocket wird nicht vom Elternprozess geerbt)
     * Das Port-Reuse muss aktiviert sein, damit keine "Address already in use"-Fehler auftreten
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Überwachung ausführen
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Worker ausführen
Worker::runAll();
```

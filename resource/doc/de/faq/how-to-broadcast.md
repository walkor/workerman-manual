# Wie man Daten broadcastet (Gruppenversand)

## Beispiel (Zeitgesteuerter Broadcast)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// In diesem Beispiel muss die Anzahl der Prozesse 1 sein
$worker->count = 1;
// Beim Start des Prozesses einen Timer einstellen, um in regelmäßigen Abständen Daten an alle Client-Verbindungen zu senden
$worker->onWorkerStart = function($worker)
{
    // Zeitgesteuert, alle 10 Sekunden
    Timer::add(10, function()use($worker)
    {
        // Gehe alle aktuellen Client-Verbindungen in diesem Prozess durch und sende die aktuelle Serverzeit
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Starte den Worker
Worker::runAll();
```

## Beispiel (Gruppenchat)

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// In diesem Beispiel muss die Anzahl der Prozesse 1 sein
$worker->count = 1;
// Beim Empfang einer Nachricht vom Client, diese an alle anderen Benutzer broadcasten
$worker->onMessage = function(TcpConnection $connection, $message)use($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// Starte den Worker
Worker::runAll();
```

## Erklärung:
**Einzelner Prozess:**
Die obigen Beispiele funktionieren nur mit einem **einzelnen Prozess** (`$worker->count=1`), da bei mehreren Prozessen mehrere Clients möglicherweise mit verschiedenen Prozessen verbunden werden und die Clients zwischen den Prozessen isoliert sind, was bedeutet, dass ein Prozess A nicht **direkt** auf das Client-Verbindungsobjekt von Prozess B zugreifen und Daten senden kann. (Um dies zu erreichen, ist eine Kommunikation zwischen den Prozessen erforderlich, beispielsweise kann die Channel-Komponente verwendet werden, wie im [Beispiel - Cluster-Sendung](../components/channel-examples.md) oder [Beispiel - Gruppen-Sendung](../components/channel-examples2.md)).

**Empfohlen wird die Verwendung von GatewayWorker:**
Das auf Workerman basierende GatewayWoker-Framework bietet eine bequemere Push-Mechanismus, einschließlich Gruppenübertragung, Broadcast usw. Es kann in mehreren Prozessen oder sogar auf mehreren Servern bereitgestellt werden. Wenn Daten an Clients gesendet werden müssen, wird die Verwendung des GatewayWorker-Frameworks empfohlen.

GatewayWorker-Handbuch unter: https://doc2.workerman.net

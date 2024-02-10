# Herzschlag

Hinweis: Langzeitverbindungen müssen mit einem Herzschlag versehen werden, da andernfalls die Verbindung aufgrund fehlender Kommunikation für längere Zeit von Routenknoten zwangsweise getrennt werden kann.

Der Herzschlag hat hauptsächlich zwei Funktionen:

1. Der Client sendet regelmäßig Daten an den Server, um zu verhindern, dass die Verbindung aufgrund langer Inaktivität von Firewalls bestimmter Knotenpunkte getrennt wird.

2. Der Server kann anhand des Herzschlags feststellen, ob der Client online ist. Wenn der Client innerhalb einer festgelegten Zeit keine Daten sendet, wird der Client als offline betrachtet. Dadurch können extreme Ereignisse wie Stromausfälle, Netzwerkausfälle usw. erkannt werden.

Empfohlenes Intervall für den Herzschlag:

Es wird empfohlen, dass der Client den Herzschlag in Intervallen von weniger als 60 Sekunden sendet, beispielsweise alle 55 Sekunden.

> Das Datenformat des Herzschlags ist nicht festgelegt, solange es vom Server erkannt werden kann.

## Beispiel für Herzschlag
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Herzschlagintervall von 55 Sekunden
define('HEARTBEAT_TIME', 55);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function(TcpConnection $connection, $msg) {
    // Temporäres Setzen einer 'lastMessageTime'-Eigenschaft für die Verbindung, um die Zeit des letzten empfangenen Nachrichten zu speichern
    $connection->lastMessageTime = time();
    // Andere Geschäftslogik...
};

// Nach dem Start des Prozesses wird ein Timer eingerichtet, der alle 10 Sekunden ausgeführt wird
$worker->onWorkerStart = function($worker) {
    Timer::add(10, function()use($worker){
        $time_now = time();
        foreach($worker->connections as $connection) {
            // Es ist möglich, dass diese Verbindung noch keine Nachricht erhalten hat, in diesem Fall wird 'lastMessageTime' auf die aktuelle Zeit gesetzt
            if (empty($connection->lastMessageTime)) {
                $connection->lastMessageTime = $time_now;
                continue;
            }
            // Wenn die Zeit zwischen den letzten Kommunikationszeiten größer als das Herzschlagintervall ist, wird angenommen, dass der Client offline ist, die Verbindung wird geschlossen
            if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                $connection->close();
            }
        }
    });
};

Worker::runAll();
```

Mit der obigen Konfiguration wird, wenn der Client länger als 55 Sekunden keine Daten an den Server sendet, der Server den Client als offline betrachten, die Verbindung schließen und `onClose` auslösen.

## Wiederverbinden bei Verbindungsabbruch (wichtig)

Unabhängig davon, ob der Client den Herzschlag sendet oder der Server den Herzschlag sendet, kann die Verbindung getrennt werden. Beispielsweise kann JavaScript im Browser pausiert werden, wenn der Browser minimiert wird oder auf eine andere Registerkarte gewechselt wird, der Computer in den Ruhezustand versetzt wird usw., bei mobilen Geräten können Netzwerke gewechselt werden, die Signalstärke kann schwächer werden, das Telefon kann gesperrt werden, Anwendungen können im Hintergrund laufen, es können Routerprobleme auftreten und es kann zu aktiven Verbindungstrennungen kommen. Besonders in komplexen Internetumgebungen werden inaktive Verbindungen oft von Routenknoten nach einer Minute aufgeräumt, was auch der Grund dafür ist, dass das Herzschlagintervall kleiner als 1 Minute sein sollte.

Verbindungen werden in Internetumgebungen leicht unterbrochen, daher ist die automatische Wiederverbindung eine unerlässliche Funktion für Langzeitverbindungen (die Wiederverbindung kann nur vom Client durchgeführt werden, der Server kann das nicht implementieren). Beispielsweise muss Websocket im Browser das `onclose`-Ereignis überwachen, um bei einem `onclose`-Ereignis eine neue Verbindung herzustellen (um Abstürze zu vermeiden). Noch strenger betrachtet sollte der Server regelmäßig den Herzschlag senden, und der Client sollte regelmäßig überwachen, ob der Herzschlag des Servers abgelaufen ist. Wenn innerhalb der vorgegebenen Zeit kein Herzschlagsignal vom Server empfangen wird, sollte die Verbindung als getrennt betrachtet werden, sie sollte geschlossen werden und eine neue Verbindung sollte hergestellt werden.


# Wie sendet man Daten an einen bestimmten Client in WorkerMan

Wenn Sie den Worker als Server verwenden, anstatt GatewayWorker zu verwenden, wie können Sie dann Nachrichten an einen bestimmten Benutzer senden?

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Initialisieren eines Worker-Containers, der den Port 1234 überwacht
$worker = new Worker('websocket://workerman.net:1234');
// ==== Die Anzahl der Prozesse muss auf 1 gesetzt werden ====
$worker->count = 1;
// Hinzufügen eines Attributs, um die Zuordnung von uid zu Verbindungen zu speichern (uid ist die Benutzer-ID oder eine eindeutige Kennung des Clients)
$worker->uidConnections = array();
// Callback-Funktion, die ausgeführt wird, wenn eine Nachricht vom Client empfangen wird
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Überprüfen, ob der aktuelle Client bereits authentifiziert ist, d.h. ob die uid festgelegt ist
    if(!isset($connection->uid))
    {
       // Wenn nicht authentifiziert, wird das erste Paket als uid behandelt (hier wird aus Gründen der Demonstration keine echte Authentifizierung durchgeführt)
       $connection->uid = $data;
       /* Speichern der Zuordnung von uid zu Verbindung. Auf diese Weise kann die Verbindung einfach über die uid gefunden werden, um spezifische Daten zu senden
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return $connection->send('Anmeldung erfolgreich, Ihre uid ist ' . $connection->uid);
    }
    // Andere Logik, um an eine bestimmte uid zu senden oder eine globale Rundfunknachricht zu senden
    // Angenommen, das Nachrichtenformat ist uid:Nachricht, dann wird die Nachricht an die uid gesendet
    // Wenn uid "all" ist, handelt es sich um eine globale Rundfunknachricht
    list($recv_uid, $message) = explode(':', $data);
    // Globale Rundfunknachricht
    if($recv_uid == 'all')
    {
        broadcast($message);
    }
    // Nachricht an eine bestimmte uid senden
    else
    {
        sendMessageByUid($recv_uid, $message);
    }
};

// Wenn ein Client die Verbindung trennt
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // Beim Trennen der Verbindung wird die Zuordnung gelöscht
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Daten an alle authentifizierten Benutzer senden
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// Daten an eine uid senden
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
    }
}

// Alle Worker ausführen (eigentlich ist nur einer definiert)
Worker::runAll();
```

**Hinweis:**

Das obige Beispiel ermöglicht das Senden an eine uid, auch wenn es nur ein Prozess ist, können problemlos 100.000 Benutzer gleichzeitig online sein.

Beachten Sie, dass in diesem Beispiel nur ein Prozess möglich ist, d.h. $worker->count muss auf 1 gesetzt sein. Um Mehrprozessunterstützung oder Server-Clusters zu ermöglichen, ist die Kommunikation zwischen den Prozessen mit dem Channel-Komponente erforderlich. Die Entwicklung ist auch sehr einfach und Sie können sich das Beispiel für die Cluster-Push-Nachrichten in der [Channel-Komponenten-Dokumentation](../components/channel-examples.md) ansehen.

**Wenn Sie Nachrichten an Clients in anderen Projekten senden möchten, können Sie sich die Anleitung in [Push-Nachrichten in anderen Projekten](push-in-other-project.md) ansehen**

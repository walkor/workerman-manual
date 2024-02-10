# Wie man asynchrone Aufgaben implementiert

**Frage:**

Wie kann man schwere Geschäftslogik asynchron verarbeiten, um zu vermeiden, dass die Hauptgeschäftslogik lange blockiert wird? Zum Beispiel, wenn ich 1000 Benutzern E-Mails senden möchte, dauert dieser Prozess sehr lange und könnte einige Sekunden blockieren. Während dieser Zeit kann die Blockierung der Hauptsequenz nachfolgende Anfragen beeinträchtigen. Wie kann man solche schwere Aufgaben an andere Prozesse zur asynchronen Verarbeitung übergeben?

**Antwort:**

Es ist möglich, auf dem lokalen Rechner oder sogar auf einem anderen Server oder Servercluster im Voraus einige Task-Prozesse für die Verarbeitung schwerer Geschäftslogik zu erstellen. Die Anzahl der Task-Prozesse kann erhöht werden, z. B. das 10-fache der CPU. Anschließend kann der Aufrufer AsyncTcpConnection verwenden, um die Daten asynchron an diese Task-Prozesse zu senden und das Ergebnis asynchron zu erhalten.

Server für Task-Prozesse:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Task-Worker, verwendet das Text-Protokoll
$task_worker = new Worker('Text://0.0.0.0:12345');
// Die Anzahl der Task-Prozesse kann je nach Bedarf erhöht werden
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function(TcpConnection $connection, $task_data)
{
     // Angenommen, es wurden JSON-Daten gesendet
     $task_data = json_decode($task_data, true);
     // Verarbeitung der entsprechenden Aufgabenlogik basierend auf task_data... Ergebnis erhalten, hier wird es ausgelassen....
     $task_result = ......
     // Ergebnis senden
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

Aufruf in Workerman:
```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// WebSocket-Server
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $ws_connection, $message)
{
    // Asynchrone Verbindung zum entfernten Task-Server herstellen, die IP-Adresse des entfernten Task-Servers, wenn es sich um den lokalen Rechner handelt, wäre 127.0.0.1, bei einem Cluster wäre es die IP-Adresse des LVS
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // Aufgaben- und Parameterdaten
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // Daten senden
    $task_connection->send(json_encode($task_data));
    // Asynchrones Ergebnis erhalten
    $task_connection->onMessage = function(AsyncTcpConnection $task_connection, $task_result)use($ws_connection)
    {
         // Ergebnis
         var_dump($task_result);
         // Nach Erhalt des Ergebnisses die asynchrone Verbindung schließen
         $task_connection->close();
         // Den entsprechenden WebSocket-Client über die abgeschlossene Aufgabe benachrichtigen
         $ws_connection->send('Aufgabe abgeschlossen');
    };
    // Asynchrone Verbindung ausführen
    $task_connection->connect();
};

Worker::runAll();
```

Auf diese Weise werden schwere Aufgaben von lokalen oder anderen Serverprozessen übernommen, und nach Abschluss der Aufgabe wird das Ergebnis asynchron empfangen, so dass die Geschäftslogik nicht blockiert wird.

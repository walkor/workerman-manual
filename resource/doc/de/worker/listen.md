# hören
```php
void Worker::listen(void)
```
Wird verwendet, um nach der Instanziierung des Workers zuzuhören.

Diese Methode dient hauptsächlich dazu, nach dem Start des Worker-Prozesses dynamisch neue Worker-Instanzen zu erstellen, die mehrere Ports in einem Prozess überwachen können und verschiedene Protokolle unterstützen. Beachten Sie, dass dies lediglich dazu dient, dem aktuellen Prozess einen weiteren Überwachungspunkt hinzuzufügen, ohne dass dynamisch neue Prozesse erstellt oder die onWorkerStart-Methode ausgelöst wird.

Wenn beispielsweise ein HTTP-Worker gestartet und dann ein WebSocket-Worker instanziiert wird, kann dieser Prozess sowohl über das HTTP- als auch das WebSocket-Protokoll zugegriffen werden. Da der WebSocket-Worker und der HTTP-Worker sich im selben Prozess befinden, können sie gemeinsame Speichervariablen nutzen und alle Socket-Verbindungen teilen. Es ist möglich, HTTP-Anfragen zu empfangen und dann die WebSocket-Clientseite zu bedienen, um beispielsweise Daten an den Client zu senden.

**Hinweis:**

Wenn die PHP-Version <= 7.0 beträgt, wird das Instantiieren eines Workers mit demselben Port in mehreren Unterprozessen nicht unterstützt. Wenn Prozess A einen Worker instanziiert, der den Port 2016 überwacht, kann Prozess B keinen Worker instanziieren, der den Port 2016 überwacht, da sonst ein Fehler ```Address already in use``` auftritt. Das folgende Beispiel würde beispielsweise ```nicht``` funktionieren:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
// 4 Prozesse
$worker->count = 4;
// Nach dem Start jedes Prozesses wird im aktuellen Prozess ein Worker für das Zuhören hinzugefügt
$worker->onWorkerStart = function($worker)
{
    /**
     * Beim Start der 4 Prozesse wird jeweils ein Worker für den Port 2016 erstellt.
     * Wenn die Methode worker->listen() ausgeführt wird, tritt der Fehler "Address already in use" auf.
     * Wenn worker->count=1 ist, tritt dieser Fehler nicht auf.
     */
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    // Das Zuhören beginnt. Hier wird der Fehler "Address already in use" auftreten
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

Wenn Ihre PHP-Version >= 7.0 ist, können Sie Worker->reusePort=true setzen, um mehrere Unterprozesse mit demselben Port für den Worker zu erstellen. Sehen Sie sich das folgende Beispiel an:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4 Prozesse
$worker->count = 4;
// Nach dem Start jedes Prozesses wird im aktuellen Prozess ein Worker für das Zuhören hinzugefügt
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    // Port-Wiederverwendung aktivieren, um Worker für denselben Port zu erstellen (erfordert PHP>=7.0)
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Das Zuhören beginnt. Es wird kein Fehler auftreten.
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

### Beispiel PHP-Backend, das Nachrichten in Echtzeit an den Client sendet

**Prinzip:**

1. Ein WebSocket-Worker wird erstellt, um die Langzeitverbindung des Clients aufrechtzuerhalten.

2. Innerhalb des WebSocket-Workers wird ein Text-Worker erstellt.

3. Der WebSocket-Worker und der Text-Worker sind im selben Prozess und können Clientverbindungen bequem gemeinsam nutzen.

4. Ein unabhängiges PHP-Backendsystem kommuniziert mit dem Text-Worker über das Text-Protokoll.

5. Der Text-Worker führt die Datenübertragung für die WebSocket-Verbindung durch.

**Code und Schritte**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Initialisierung eines Worker-Containers, der den Port 1234 überwacht
$worker = new Worker('websocket://0.0.0.0:1234');

/*
 * Beachten Sie, dass die Anzahl der Prozesse hier auf 1 festgelegt sein muss
 */
$worker->count = 1;
// Nach dem Start des Worker-Prozesses wird ein Text-Worker erstellt, um einen internen Kommunikationsport zu öffnen
$worker->onWorkerStart = function($worker)
{
    // Öffnen eines internen Ports, um die interne Datenübertragung zu ermöglichen. Text-Protokollformat: Text + Zeilenumbruch
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        // Das Datenarrangement enthält eine uid, die angibt, an welche uid die Daten gesendet werden sollen
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // Mit Workerman können Daten an die Seite mit der uid gesendet werden
        $ret = sendMessageByUid($uid, $buffer);
        // Ergebnis der Datenübertragung zurückgeben
        $connection->send($ret ? 'ok' : 'fail');
    };
    // ## Zuhören beginnt ##
    $inner_text_worker->listen();
};
// Hinzufügen einer Eigenschaft zum Speichern der Zuordnung von uid zu Verbindung
$worker->uidConnections = array();
// Ausführung der Callback-Funktion, wenn eine Nachricht von einem Client empfangen wird
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Überprüfen, ob der aktuelle Client bereits überprüft wurde, d. h. ob eine uid festgelegt wurde
    if(!isset($connection->uid))
    {
       // Falls nicht überprüft wurde, wird das erste Paket als uid behandelt (hier aus Demonstrationsgründen wurden keine tatsächlichen Überprüfungen durchgeführt)
       $connection->uid = $data;
       /* Die Zuordnung von uid zu Verbindung speichern, um die Verbindung anhand der uid leicht finden und Daten spezifisch an eine uid übertragen zu können */
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

// Callback-Funktion wird ausgeführt, wenn ein Client die Verbindung trennt
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // Beim Trennen der Verbindung wird die Zuordnung gelöscht
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Datenübertragung an alle überprüften Benutzer
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// Datenübertragung an eine bestimmte uid
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// Ausführen aller Worker
Worker::runAll();
```

Starten Sie den Backend-Service
```php push.php start -d```

JavaScript-Code zum Empfangen von Push-Nachrichten auf der Clientseite
```javascript
var ws = new WebSocket('ws://127.0.0.1:1234');
ws.onopen = function(){
    var uid = 'uid1';
    ws.send(uid);
};
ws.onmessage = function(e){
    alert(e.data);
};
```

Code zum Senden von Push-Nachrichten vom Backend
```php
// Eine Socket-Verbindung zum internen Push-Port herstellen
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
// Die zu sendenden Daten, einschließlich des uid-Felds, das angibt, an diese uid gesendet werden soll
$data = array('uid'=>'uid1', 'percent'=>'88%');
// Daten senden, beachten Sie, dass der Port 5678 ein Text-Port ist und das Text-Protokoll ein Zeilenumbruch am Ende der Daten erfordert
fwrite($client, json_encode($data)."\n");
// Push-Ergebnis lesen
echo fread($client, 8192);
```

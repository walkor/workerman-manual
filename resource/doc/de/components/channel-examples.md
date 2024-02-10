```php
# Beispiel 1
**``` (Erfordert Workerman-Version >= 3.3.0) ```**

Mehrpunkt-Worker-basiertes (verteiltes Cluster) Push-System, Cluster-Weiterleitung, Cluster-Broadcast.

`start_channel.php`
Nur ein start_channel-Dienst kann im gesamten System bereitgestellt werden. Angenommen, er läuft auf 192.168.1.1.
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Initialisieren Sie einen Channel-Server
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```

`start_ws.php`
Es können mehrere start_ws-Dienste im gesamten System bereitgestellt werden, angenommen auf den beiden Servern 192.168.1.2 und 192.168.1.3.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Websocket-Server
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // Client verbindet sich mit dem Channel-Server
    Channel\Client::connect('192.168.1.1', 2206);
    // Verwenden der eigenen Prozess-ID als Ereignisnamen
    $event_name = $worker->id;
    // Abonnieren des Ereignisses worker->id und Registrieren der Ereignisbehandlungsfunktion
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "Verbindung existiert nicht\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // Abonnieren des Broadcast-Ereignisses
    $event_name = 'Broadcast';
    // Senden von Broadcast-Daten an alle Client-Verbindungen im aktuellen Prozess nach Erhalt des Broadcast-Ereignisses
    Channel\Client::on($event_name, function($event_data)use($worker){
        $message = $event_data['content'];
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    });
};

$worker->onConnect = function(TcpConnection $connection)use($worker)
{
    $msg = "Worker-ID:{$worker->id} Verbindungs-ID:{$connection->id} verbunden\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```

`start_http.php`
Es können mehrere start_ws-Dienste im gesamten System bereitgestellt werden, angenommen auf den beiden Servern 192.168.1.4 und 192.168.1.5.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Zur Verarbeitung von HTTP-Anfragen, um Daten an beliebige Clients zu senden, sind workerID und connectionID erforderlich
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // Kompatibel mit workerman 4.x
    if (!is_array($request)) {
            $_GET = $request->get();
   }
    $connection->send('ok');
    if(empty($_GET['content'])) return;
    // Daten an bestimmten Worker-Prozess und bestimmte Verbindung senden
    if(isset($_GET['to_worker_id']) && isset($_GET['to_connection_id']))
    {
        $event_name = $_GET['to_worker_id'];
        $to_connection_id = $_GET['to_connection_id'];
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'to_connection_id' => $to_connection_id,
           'content'          => $content
        ));
    }
    // Globale Broadcast-Daten
    else
    {
        $event_name = 'Broadcast';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};

Worker::runAll();
```

## Testen
1. Führen Sie die Dienste auf den einzelnen Servern aus

2. Client verbindet sich mit dem Server

Öffnen Sie den Chrome-Browser, drücken Sie F12, um die Entwicklertools zu öffnen, geben Sie im Konsolenfenster ein (oder fügen Sie den untenstehenden Code in eine HTML-Seite ein und führen Sie ihn mit JavaScript aus)

```javascript
// Sie können auch ws://192.168.1.3:4236 verwenden
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("Nachricht vom Server erhalten: " + e.data);
};
```

3. Push über den Aufruf des HTTP-Interfaces

Rufen Sie die URL ```http://192.168.1.4:4237/?content={$content}``` oder ```http://192.168.1.5:4237/?content={$content}``` auf, um die Daten ```$content``` an alle Client-Verbindungen zu senden.

Rufen Sie die URL ```http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` oder ```http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` auf, um die Daten ```$content``` an eine bestimmte Client-Verbindung in einem bestimmten Worker-Prozess zu senden.

Hinweis: Ersetzen Sie zum Testen ```{$worker_id}``` ```{$connection_id}``` und ```{$content}``` durch die tatsächlichen Werte.
```

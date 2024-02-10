# Beispiel 1
**``` (Workerman-Version> = 3.3.0 erforderlich) ```**

Mehrprozess-Gruppen-Push-System basierend auf dem Worker

``` php
    use Workerman\Worker;
    use Workerman\Connection\TcpConnection;
    require_once __DIR__ . '/vendor/autoload.php';

    $channel_server = new Channel\Server('0.0.0.0', 2206);

    $worker = new Worker('websocket://0.0.0.0:1234');
    $worker->count = 8;
    // Globale Zuordnung von Gruppen zu Verbindungen
    $group_con_map = array();
    $worker->onWorkerStart = function(){
        // Channel-Client verbindet sich mit dem Channel-Server
        Channel\Client::connect('127.0.0.1', 2206);

        // Überwacht das Ereignis zum Versenden von Nachrichten an globale Gruppen
        Channel\Client::on('send_to_group', function($event_data){
            $group_id = $event_data['group_id'];
            $message = $event_data['message'];
            global $group_con_map;
            var_dump(array_keys($group_con_map));
            if (isset($group_con_map[$group_id])) {
                foreach ($group_con_map[$group_id] as $con) {
                    $con->send($message);
                }
            }
        });
    };
    $worker->onMessage = function(TcpConnection $con, $data){
        // Nachricht zum Beitritt zu einer Gruppe {"cmd":"add_group", "group_id":"123"}
        // oder Massennachricht {"cmd":"send_to_group", "group_id":"123", "message":"Dies ist eine Nachricht"}
        $data = json_decode($data, true);
        var_dump($data);
        $cmd = $data['cmd'];
        $group_id = $data['group_id'];
        switch($cmd) {
            // Verbindung tritt Gruppe bei
            case "add_group":
                global $group_con_map;
                // Fügt die Verbindung zum entsprechenden Gruppenarray hinzu
                $group_con_map[$group_id][$con->id] = $con;
                // Speichert, zu welchen Gruppen diese Verbindung gehört, damit sich beim Schließen die Gruppendaten in group_con_map leicht bereinigen lassen
                $con->group_id = isset($con->group_id) ? $con->group_id : array();
                $con->group_id[$group_id] = $group_id;
                break;
            // Massennachricht an die Gruppe senden
            case "send_to_group":
                // Channel-Client sendet an alle Serverprozesse das Ereignis zum Gruppenversand
                Channel\Client::publish('send_to_group', array(
                    'group_id'=>$group_id,
                    'message'=>$data['message']
                ));
                break;
        }
    };
    // Wichtig: Bei Verbindungsabbruch die Verbindung aus den globalen Gruppendaten entfernen, um Speicherlecks zu vermeiden
    $worker->onClose = function(TcpConnection $con){
        global $group_con_map;
        // Durchläuft alle Gruppen, zu denen die Verbindung gehört, und entfernt die entsprechenden Daten aus group_con_map
        if (isset($con->group_id)) {
            foreach ($con->group_id as $group_id) {
                unset($group_con_map[$group_id][$con->id]);
                if (empty($group_con_map[$group_id])) {
                    unset($group_con_map[$group_id]);
                }
            }
        }
    };

    Worker::runAll();
```

## Testen (Angenommen, alle laufen auf der lokalen 127.0.0.1)
1. Starten des Servers
```php
php start.php start
Workerman[del.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.4.2          PHP version:7.1.3
------------------------ WORKERS -------------------------------
user          worker         listen                    processes status
liliang       ChannelServer  frame://0.0.0.0:2206       1         [OK] 
liliang       none           websocket://0.0.0.0:1234   12        [OK] 
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

2. Client verbindet sich mit dem Server

Öffnen Sie den Chrome-Browser, drücken Sie F12, um die Entwicklertools zu öffnen, und geben Sie im Konsolenfenster Folgendes ein (oder fügen Sie den folgenden Code in eine HTML-Seite ein und führen Sie ihn mit JavaScript aus)

```javascript
// Angenommen die Server-IP ist 127.0.0.1, ändern Sie diese für den Test in die tatsächliche Server-IP
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
	ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"Dies ist eine Nachricht"}');
};

```

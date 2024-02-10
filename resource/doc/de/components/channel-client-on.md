```php
# on
**``` (Workerman-Version >=3.3.0 erforderlich) ```**
```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```
Abonniere das Ereignis ```$event_name``` und registriere die Callback-Funktion ```$callback_function```, die auftritt, wenn das Ereignis eintritt.

## Parameter der Callback-Funktion

 ``` $event_name ```

Der Name des abonnierten Ereignisses, der beliebig sein kann.

 ``` $callback_function ```

Die Callback-Funktion, die aufgerufen wird, wenn das Ereignis eintritt. Die Prototyp der Funktion ist ```callback_function(mixed $event_data)```. ```$event_data``` ist die Ereignisdaten, die beim Veröffentlichen des Ereignisses übergeben wurden.

Beachte:

Wenn dieselbe Veranstaltung zwei Callback-Funktionen registriert, wird die zweite Callback-Funktion die erste überschreiben.


## Beispiel
Multiprozess-Worker (Mehrere Server), ein Client sendet eine Nachricht an alle Clients.


start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Initialize a Channel server
$channel_server = new Channel\Server('0.0.0.0', 2206);

// WebSocket Server
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->name = 'websocket';
$worker->count = 6;
// When each worker process starts
$worker->onWorkerStart = function($worker)
{
    // Channel client connects to the Channel server
    Channel\Client::connect('127.0.0.1', 2206);
    // Subscribe to broadcast event and register event callback
    Channel\Client::on('broadcast', function($event_data)use($worker){
        // Broadcast message to all client connections of the current worker process
        foreach($worker->connections as $connection)
        {
            $connection->send($event_data);
        }
    });
};

$worker->onMessage = function(TcpConnection $connection, $data)
{
   // Treat the data sent by the client as event data
   $event_data = $data;
   // Publish broadcast event to all worker processes
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```

**Test**

Open Chrome browser, press F12 to open the debugging console, and enter the following code in the Console column (or put the following code into an HTML page to run with JavaScript)

Receive message connection
```javascript
// Replace 127.0.0.1 with the actual Workerman IP
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("Received message from the server: " + e.data);
};
```

Broadcast message
```
ws.send('hello world');
```

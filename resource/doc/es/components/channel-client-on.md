# on
**``` (Requires Workerman version>=3.3.0) ```**
```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```
Subscribes to the ```$event_name``` event and registers the callback ```$callback_function``` to be executed when the event occurs.

## Parameters of the callback function

 ``` $event_name ```

The name of the subscribed event, which can be any string.

 ``` $callback_function ```

The callback function triggered when the event occurs. The function prototype is ```callback_function(mixed $event_data)```. ```$event_data``` is the event data passed when the event is published.

Note:

If two callback functions are registered for the same event, the latter one will override the former one.

## Example
Multiprocess Worker (multiple servers), where a client sends a message and it is broadcast to all clients.

start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Initialize a Channel server
$channel_server = new Channel\Server('0.0.0.0', 2206);

// WebSocket server
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->name = 'websocket';
$worker->count = 6;
// When each worker process starts
$worker->onWorkerStart = function($worker)
{
    // Connect Channel client to the Channel server
    Channel\Client::connect('127.0.0.1', 2206);
    // Subscribe to the broadcast event and register the event callback
    Channel\Client::on('broadcast', function($event_data) use ($worker){
        // Broadcast the message to all clients of the current worker process
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
   // Publish the broadcast event to all worker processes
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```

**Testing**

Open the Chrome browser, press F12 to open the developer tools, and in the Console tab, type the following (or integrate the code into an HTML page and run it using JavaScript).

Connections receiving messages
```javascript
// Replace 127.0.0.1 with the actual IP where Workerman is running
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("Received message from the server: " + e.data);
};
```

Broadcasting a message
```
ws.send('hello world');
```

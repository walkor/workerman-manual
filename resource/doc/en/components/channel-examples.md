# Example 1
**``` (requires Workerman version >= 3.3.0) ```**

Worker-based multi-process (distributed cluster) push system, cluster broadcast, and cluster-wide broadcasting.

`start_channel.php`
Only one `start_channel` service can be deployed in the entire system. Assuming it runs on 192.168.1.1.
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Initialize a Channel server
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```

`start_ws.php`
Multiple `start_ws` services can be deployed in the entire system, assuming running on two servers at 192.168.1.2 and 192.168.1.3.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Websocket server
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // Connect to Channel server as a Client
    Channel\Client::connect('192.168.1.1', 2206);
    // Use its own process id as the event name
    $event_name = $worker->id;
    // Subscribe to the worker's id event and register the event handling function
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "connection not exists\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // Subscribe to the broadcast event
    $event_name = 'broadcast';
    // Upon receiving the broadcast event, send broadcast data to all client connections within the current process
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
    $msg = "workerID:{$worker->id} connectionID:{$connection->id} connected\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```

`start_http.php`
Multiple `start_ws` services can be deployed in the entire system, assuming running on two servers at 192.168.1.4 and 192.168.1.5.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Used to handle HTTP requests, push data to any client, and require workerID and connectionID
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // Compatible with workerman 4.x
    if (!is_array($request)) {
            $_GET = $request->get();
   }
    $connection->send('ok');
    if(empty($_GET['content'])) return;
    // It is to push data to a certain worker process and a certain connection within it
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
    // It is a global broadcast of data
    else
    {
        $event_name = 'broadcast';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};

Worker::runAll();
```

## Testing
1. Run the services on each server

2. Connect the client to the server

Open the Chrome browser, press F12 to open the Developer Tools, and in the Console section, input the following (or put the code below into an HTML page to run with JavaScript)

```javascript
// ws://192.168.1.3:4236 can also be used
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("Received message from the server: " + e.data);
};
```

3. Push through the HTTP interface

Access the URL `http://192.168.1.4:4237/?content={$content}` or `http://192.168.1.5:4237/?content={$content}` to push `$content` data to all client connections.

Access the URL `http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}` or `http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}` to push `$content` data to a certain client connection within a certain worker process.

Note: Replace `{$worker_id}`, `{$connection_id}`, and `{$content}` with actual values during testing.

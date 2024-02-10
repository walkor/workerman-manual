```php
void Worker::listen(void)
```
Used to instantiate Worker and execute the listen function.

This method is mainly used to dynamically create new Worker instances after the Worker process starts, which can achieve listening on multiple ports in the same process and support multiple protocols. It should be noted that using this method only adds listening in the current process, does not create new processes dynamically, and does not trigger the onWorkerStart method.

For example, after starting an HTTP Worker, if you instantiate a WebSocket Worker, then this process can be accessed via both the HTTP protocol and the WebSocket protocol. Since the WebSocket Worker and HTTP Worker are in the same process, they can access shared memory variables and share all socket connections. This can achieve the effect of receiving HTTP requests and then interacting with WebSocket clients to push data to clients.

**Note:**

If PHP version <= 7.0, it does not support instantiating Workers with the same port in multiple child processes. For example, if process A creates a Worker listening on port 2016, process B cannot create a Worker listening on port 2016, otherwise an "Address already in use" error will be reported. The following code will ```not``` run.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
// 4 processes
$worker->count = 4;
// After each process starts, instantiate a new Worker to listen in the current process
$worker->onWorkerStart = function($worker)
{
    /**
     * When the 4 processes start, they all create a Worker listening on port 2016
     * When executing worker->listen(), an Address already in use error will be reported
     * If worker->count=1, it will not report an error
     */
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    // Execute listen. An "Address already in use" error will be reported here
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Run worker
Worker::runAll();
```

If your PHP version >= 7.0, you can set Worker->reusePort=true, which allows multiple child processes to create Workers with the same port. See the example below:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4 processes
$worker->count = 4;
// After each process starts, instantiate a new Worker to listen in the current process
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    // Set port reuse, can create Workers listening on the same port (requires PHP>=7.0)
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Execute listen. No error will be reported for normal listening
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Run worker
Worker::runAll();
```



### Example of pushing messages to clients in real time with a PHP backend

**Principle:**

1. Establish a WebSocket Worker to maintain long connections with clients
2. Build a text Worker inside the WebSocket Worker
3. The WebSocket Worker and the text Worker are in the same process, allowing for easy sharing of client connections
4. An independent PHP backend system communicates with the text Worker via the text protocol
5. The text Worker operates WebSocket connections to push data

**Code and steps:**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Initialize a worker container, listening on port 1234
$worker = new Worker('websocket://0.0.0.0:1234');

/*
 * Note that the number of processes must be set to 1 here
 */
$worker->count = 1;
// Create a text Worker after the worker process starts in order to open an internal communication port
$worker->onWorkerStart = function($worker)
{
    // Open an internal port for easy data push from internal systems, Text protocol format: text + newline character
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        // The $data array format contains uid, indicating which uid's page is to receive the data push
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // Use Workerman to push data to the uid's page
        $ret = sendMessageByUid($uid, $buffer);
        // Return the push result
        $connection->send($ret ? 'ok' : 'fail');
    };
    // ## Execute listen ##
    $inner_text_worker->listen();
};
// Add a new property to save the mapping of uid to connection
$worker->uidConnections = array();
// Callback function when a client sends a message
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Check if the current client has been authenticated, i.e., if uid has been set
    if (!isset($connection->uid))
    {
       // If not authenticated, use the first package as the uid (for demonstration purposes, no real verification is done here)
       $connection->uid = $data;
       /* Save the mapping of uid to connection, making it easy to find the connection via uid and push data for a specific uid */
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

// When a client connection is closed
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if (isset($connection->uid))
    {
        // Delete the mapping when the connection is closed
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Push data to all authenticated users
function broadcast($message)
{
    global $worker;
    foreach ($worker->uidConnections as $connection)
    {
        $connection->send($message);
    }
}

// Push data for a specific uid
function sendMessageByUid($uid, $message)
{
    global $worker;
    if (isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// Run all workers
Worker::runAll();
```

Start the backend service
```php push.php start -d```

JavaScript code to receive pushes from the frontend

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

Backend code to push messages

```php
// Connect to the internal push port via socket
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
// Data to be pushed, including the uid field, indicating the uid to which the data should be pushed
$data = array('uid'=>'uid1', 'percent'=>'88%');
// Send data, note that port 5678 is the Text protocol port, which requires adding a newline character at the end of the data
fwrite($client, json_encode($data)."\n");
// Read the push result
echo fread($client, 8192);
```

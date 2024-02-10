# How to send data to a specific client in Workerman
When using `Worker` as the server without using `GatewayWorker`, how to push messages to a specific user?

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Initialize a worker container, listening on port 1234
$worker = new Worker('websocket://workerman.net:1234');
// ====The number of processes must be set to 1====
$worker->count = 1;
// Add a new property to save the mapping of uid to connection (uid is the user id or the unique identifier of the client)
$worker->uidConnections = array();
// Callback function to execute when a client sends a message
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Check if the current client has been validated, i.e., if the uid has been set
    if(!isset($connection->uid))
    {
       // If not validated, the first package is treated as the uid (not a real verification for demonstration purpose)
       $connection->uid = $data;
       /* Save the mapping of uid to connection, so that connections can be easily found by uid,
        * and implement targeted data push for a specific uid
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return $connection->send('login success, your uid is ' . $connection->uid);
    }
    // Other logic, sending to a specific uid or broadcasting globally
    // If the message format is uid:message, it will be sent to the uid
    // If uid is 'all', it will be broadcast to all
    list($recv_uid, $message) = explode(':', $data);
    // Global broadcast
    if($recv_uid == 'all')
    {
        broadcast($message);
    }
    // Send to a specific uid
    else
    {
        sendMessageByUid($recv_uid, $message);
    }
};

// When a client connection is closed
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // Delete the mapping when the connection is closed
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Push data to all validated users
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// Push data to a specific uid
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
    }
}

// Run all workers (actually only one is defined currently)
Worker::runAll();
```
**Note:**

The above example can push messages to specific uids, and although it is a single process, it can support up to 100,000 online users.

Please note that this example can only have a single process, meaning `$worker->count` must be set to 1. To support multiple processes or server clusters, the Channel component needs to be used for inter-process communication. The development is also very simple and can be referred to in the section [Channel component cluster push example](../components/channel-examples.md).

**If you want to push messages to clients in other systems, you can refer to the section [Push in other projects](push-in-other-project.md)**

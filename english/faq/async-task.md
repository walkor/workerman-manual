# How to Implement Asynchronous Tasks

**Question:**

How to handle heavy business asynchronously, avoiding the main business being blocked for a long time? For example, I need to send emails to 1000 users, which is a slow process and may block for several seconds. During this process, the main process being blocked will affect subsequent requests. How can such heavy tasks be handed over to other processes for asynchronous processing?

**Answer:**

You can pre-establish some task processes to handle heavy business on the local machine or other servers, even server clusters. The number of task processes can be increased, for example, ten times the CPU. Then the caller uses AsyncTcpConnection to asynchronously send data to these task processes for asynchronous processing and to asynchronously receive the processing results.

Task process on the server side:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Task worker, using the Text protocol
$task_worker = new Worker('Text://0.0.0.0:12345');
// The number of task processes can be increased as needed
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function(TcpConnection $connection, $task_data)
{
    // Assume that the received data is in JSON format
    $task_data = json_decode($task_data, true);
    // Process the corresponding task logic based on task_data .... Obtain the result. Omitted here.
    $task_result = ......;
    // Send the result
    $connection->send(json_encode($task_result));
};
Worker::runAll();
```

Calling in workerman:

```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// WebSocket service
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $ws_connection, $message)
{
    // Establish an asynchronous connection with the remote task service. The IP is the IP of the remote task service, 127.0.0.1 if it is local, or the IP of the LVS if it is a cluster.
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // Task and parameter data
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // Send data
    $task_connection->send(json_encode($task_data));
    // Asynchronously receive the result
    $task_connection->onMessage = function(AsyncTcpConnection $task_connection, $task_result) use ($ws_connection)
    {
        // Result
        var_dump($task_result);
        // After receiving the result, be sure to close the asynchronous connection
        $task_connection->close();
        // Notify the corresponding WebSocket client that the task is complete
        $ws_connection->send('task complete');
    };
    // Execute the asynchronous connection
    $task_connection->connect();
};

Worker::runAll();
```

In this way, the heavy tasks are handed over to processes on the local machine or other servers to handle, and the results are received asynchronously after the task is completed, so the business process will not be blocked.

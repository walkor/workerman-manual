# Basic Process
(Using a simple Websocket chat room server as an example)

#### 1. Establish project directory at any location
For example, SimpleChat/
Go to the directory and execute `composer require workerman/workerman`

#### 2. Include `vendor/autoload.php` (generated after composer installation)
Create start.php and include `vendor/autoload.php`
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3. Define the protocol
Here we select the Text protocol (a custom protocol in Workerman, with the format of text + newline)

(Currently Workerman supports HTTP, Websocket, and Text protocols. If you need to use other protocols, please refer to the protocol chapter to develop your own protocol)

#### 4. Write the entrance startup script as needed
For example, the following is a simple entry file for a chat room.

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// Assign a uid to the connection when a client connects, save the connection, and notify all clients
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // Assign a uid to this connection
    $connection->uid = ++$global_uid;
}

// Forward the message to everyone when a client sends a message
function handle_message(TcpConnection $connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] said: $data");
    }
}

// Broadcast to all clients when a client disconnects
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] logout");
    }
}

// Create a Worker with text protocol listening on port 2347
$text_worker = new Worker("text://0.0.0.0:2347");

// Start only 1 process for easier data transmission between clients
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();
```

#### 5. Testing
The Text protocol can be tested using the telnet command
```shell
telnet 127.0.0.1 2347
```

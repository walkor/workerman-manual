# Heartbeat

Note: Long connection applications must have a heartbeat, otherwise the connection may be forcibly disconnected by routing nodes due to long periods of inactivity.

The main purposes of a heartbeat are:

1. The client sends periodic data to the server to prevent the connection from being closed by certain firewall nodes due to lack of communication for a long time.

2. The server can determine if the client is online by using the heartbeat. If the client does not send any data within a specified time, the server will consider the client offline. This can help detect situations where the client goes offline due to extreme conditions such as power outages or network disconnections.

Recommended heartbeat interval:

It is recommended that the client sends a heartbeat interval of less than 60 seconds, such as 55 seconds.

> There are no specific requirements for the format of the heartbeat data, as long as the server can recognize it.

## Heartbeat Example
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Heartbeat interval is 55 seconds
define('HEARTBEAT_TIME', 55);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function(TcpConnection $connection, $msg) {
    // Temporarily set a 'lastMessageTime' property for the connection to record the time of the last received message
    $connection->lastMessageTime = time();
    // Other business logic...
};

// Set a timer to run every 10 seconds after the process starts
$worker->onWorkerStart = function($worker) {
    Timer::add(10, function() use ($worker) {
        $time_now = time();
        foreach ($worker->connections as $connection) {
            // If the connection has not received any messages, set 'lastMessageTime' to the current time
            if (empty($connection->lastMessageTime)) {
                $connection->lastMessageTime = $time_now;
                continue;
            }
            // If the time interval since the last communication is greater than the heartbeat interval, the client is considered offline, and the connection is closed
            if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                $connection->close();
            }
        }
    });
};

Worker::runAll();
```

In the above configuration, if the client does not send any data to the server for more than 55 seconds, the server considers the client to be offline, closes the connection, and triggers 'onClose'.

## Reconnection After Disconnection (Important)

Regardless of whether the client sends a heartbeat or the server sends a heartbeat, the connection may be disconnected. For example, JavaScript is paused when the browser is minimized, switched to other tab pages, the computer goes to sleep, network changes on mobile devices, the signal weakens, the phone screen is off, the mobile app switches to the background, router failures, or active disconnection by the business. Especially in a complex external network environment, many routing nodes will clean up inactive connections within 1 minute, which is why the recommended heartbeat interval is less than 1 minute.

Connections are easily disconnected in an external network environment, so reconnecting after disconnection is a function that long connection applications must have (reconnection after disconnection can only be done by the client, it cannot be implemented by the server). For example, in a browser-based WebSocket, listening to the 'onclose' event and establishing a new connection when 'onclose' occurs (to avoid the need for a hard break to establish a connection). Strictly speaking, the server should also periodically send heartbeat data, and the client needs to periodically check if the server's heartbeat data has timed out. If no heartbeat data from the server is received within the specified time, the connection should be considered as disconnected, requiring a close and the establishment of a new connection.

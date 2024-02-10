# send
## Description
```php
mixed Connection::send(mixed $data [,$raw = false])
```

Send data to the client.

## Parameters

``` $data ```

The data to be sent. If a protocol is specified when initializing the Worker class, the protocol's encode method will be automatically called to complete the protocol packing and send it to the client.

``` $raw ```

Whether to send raw data, i.e., not calling the protocol's encode method. The default is false, which means the protocol's encode method will be called automatically.

## Return value

true indicates that the data has been successfully written to the socket send buffer of the connection's operating system layer.

null indicates that the data has been written to the application layer send buffer of the connection, waiting to be written to the socket send buffer of the system layer.

false indicates that the sending failed, possibly because the client connection has been closed, or the application layer send buffer of the connection is full.

## Note
If send returns ```true```, it only means that the data has been successfully written to the socket send buffer of the connection’s operating system, but does not imply that the data has been successfully sent to the receiving socket buffer of the peer, let alone that the peer application has read the data from the local socket receive buffer. **However, as long as send does not return false and the network is not disconnected, and the client receives normally, the data can basically be regarded as being delivered to the other party.**

Since the data in the socket send buffer is asynchronously sent to the peer by the operating system, the operating system does not provide a corresponding confirmation mechanism to the application layer. Therefore, **the application layer** cannot know when the data in the socket send buffer starts to be sent, and **the application layer** cannot know whether the data in the socket send buffer has been successfully sent. For these reasons, Workerman cannot directly provide a message confirmation interface.

If the business needs to ensure that each message is received by the client, a confirmation mechanism can be added in the business logic. The confirmation mechanism may vary depending on the specific business requirements, and even for the same business, the confirmation mechanism can have multiple methods.

For example, a chat system can use this confirmation mechanism. Store each message in the database, and each message has a field indicating whether it has been read. When the client receives a message, it sends a confirmation packet to the server, and the server marks the corresponding message as read. When the client connects to the server (usually when the user logs in or reconnects after disconnection), it queries the database for unread messages, and if there are any, sends them to the client. Similarly, when the client receives a message, it notifies the server that it has been read. This way, it can ensure that each message is received by the other party. Of course, developers can also use their own confirmation logic.

## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // The \Workerman\Protocols\Websocket::encode method will be automatically called to pack the data into websocket protocol data and send it
    $connection->send("hello\n");
};
// Run the worker
Worker::runAll();
```

# SSE
**This feature requires workerman>=4.0.0**

SSE, also known as Server-sent Events, is a server push technology. Its essence is that after the client sends an HTTP request with the `Accept: text/event-stream` header, the connection is not closed, and the server can continuously push data to the client on this connection.

The difference between SSE and WebSocket is:
*   SSE only supports server-to-client push; WebSocket supports two-way communication.
*   SSE supports automatic reconnection by default; WebSocket needs to be implemented manually.
*   SSE can only transmit UTF-8 text, and binary data needs to be encoded as UTF-8 before transmission; WebSocket supports transmission of both UTF-8 and binary data by default.
*   SSE comes with message types; WebSocket needs to be implemented manually.

### Example
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\ServerSentEvents;
use Workerman\Protocols\Http\Response;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function (TcpConnection $connection, Request $request) {
    // If the Accept header is text/event-stream, it is an SSE request
    if ($request->header('accept') === 'text/event-stream') {
        // Firstly, send a response with Content-Type: text/event-stream header
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // Periodically push data to the client
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id) {
            // When the connection is closed, delete the timer to avoid accumulation of timers causing memory leaks
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // Send the message event with data as 'hello' and an optional message ID
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id' => 1]));
        });
        return;
    }
    $connection->send('ok');
};

// Run the worker
Worker::runAll();
```

Client-side JavaScript code
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // Outputs hello
}, false);
```

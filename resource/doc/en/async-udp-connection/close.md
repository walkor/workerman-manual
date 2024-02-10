```php
void Connection::close(mixed $data = '')
```

Safely close the connection and trigger the `onClose` callback of the connection.

Although UDP is connectionless, the corresponding AsyncUdpConnection object is always retained in the memory. You must call the `close` method to release the corresponding UDP connection object; otherwise, this UDP connection object will remain in memory, causing memory leaks.

## Parameters

``` $data ```

Optional parameter, the data to be sent (if a protocol is specified, the protocol's encode method will be automatically called to package the `$data` data). After the data is sent, the connection is closed, and then the `onClose` callback will be triggered.

The size of the data cannot exceed 65507 bytes; otherwise, the send will fail.

### Example

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // Start a UDP client 1 second later, connect to port 1234, and send the string 'hi'
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Receive the data 'hello' returned from the server
            echo "recv $data\r\n";
            // Close the connection
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Receive the data sent by the AsyncUdpConnection client and return the string 'hello'
    $connection->send("hello");
};
Worker::runAll();             
```

After execution, it will print something like:
```
recv hello
```

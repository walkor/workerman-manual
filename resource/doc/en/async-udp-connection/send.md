# send method
```php
void AsyncUdpConnection::send(string $data)
```
Performs an asynchronous connection operation. This method returns immediately.

### Parameters
``` $data ```
The data to be sent to the server. The data size cannot exceed 65507 bytes (the maximum transmission size of a UDP single data packet is 65507 bytes), otherwise the sending will fail.

### Return
No return value

### Example

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // Start a UDP client after 1 second, connect to port 1234 and send the string 'hi'
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
    // Receive the data sent by the AsyncUdpConnection client, and return the string 'hello'
    $connection->send("hello");
};
Worker::runAll();             
```

After execution, it will print something like:
```
recv hello
```

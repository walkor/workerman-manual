# __construct method
```php
void AsyncUdpConnection::__construct(string $remote_address)
```

Create a UDP connection object.

AsyncUdpConnection allows Workerman to transmit UDP data as a client to a remote server.

## Parameters
Parameter: ```remote_address```

The address of the connection, for example:
```udp://192.168.1.1:1234```
```frame://192.168.1.1:8080```
```text://192.168.1.1:8080```

## Examples
```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // Start a UDP client after 1 second, connect to port 1234, and send the string "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // Receive data "hello" returned by the server
            echo "recv $data\r\n";
            // Close the connection
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
{
    // Receive data sent by the AsyncUdpConnection client and return the string "hello"
    $connection->send("hello");
};
Worker::runAll();             
```

After execution, it prints something similar to:
```
recv hello
```

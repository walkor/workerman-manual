# Transport Property
*Requires (workerman >= 3.3.4)*

Set the transport property, with optional values of [tcp](https://baike.baidu.com/subview/32754/8048820.htm) and [ssl](https://baike.baidu.com/view/525499.htm), default is tcp.

When transport is [ssl](https://baike.baidu.com/view/525499.htm), PHP must have the [openssl extension](https://php.net/manual/zh/book.openssl.php) installed.

When using Workerman as a client to initiate an SSL-encrypted connection to the server (https or wss connection), please set this option to `ssl`, as shown in the example below.

### Example (https connection)
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Establish an asynchronous connection to www.baidu.com and send data when the process starts
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // Set up an SSL-encrypted connection
    $connection_to_baidu->transport = 'ssl';

    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "connect success\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "connection closed\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Error code:$code msg:$msg\n";
    };
    $connection_to_baidu->connect();
};

// Run the worker
Worker::runAll();
```

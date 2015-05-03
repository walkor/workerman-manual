# __construct
## Description:
```php
void \Workerman\Connection\AsyncTcpConnection::__construct(string $remote_address)
```
Create an async connection.

### Parameters
``` remote_address ```

Address，such as ```tcp://www.google.com:80```


### Examples
```php
use \Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
require_once './Workerman/Autoloader.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    $connection_to_google = new AsyncTcpConnection('tcp://www.google.com:80');

    $connection_to_google->onConnect = function($connection_to_google)
    {
        echo "connect success\n";
        $connection_to_google->send("GET / HTTP/1.1\r\nHost: www.google.com\r\nConnection: keep-alive\r\n\r\n");
    };

    $connection_to_google->onMessage = function($connection_to_google, $http_buffer)
    {
        echo $http_buffer;
    };

    $connection_to_google->onClose = function($connection_to_google)
    {
        echo "connection closed\n";
    };

    $connection_to_google->onError = function($connection_to_google, $code, $msg)
    {
        echo "Error code:$code msg:$msg\n";
    };

    $connection_to_google->connect();
};

// Run all workers
Worker::runAll();

```


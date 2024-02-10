# reConnect Method
```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```

``` (Requires Workerman version>=3.3.5) ```

Reconnect. Generally called in the `onClose` callback to achieve reconnection after the connection is dropped.

This method can be called to reconnect in case of disconnection due to network issues or the service restart of the other party.

### Parameters
 ``` $delay ```

The delay before reconnecting. The unit is seconds, supports decimals, and can be precise to milliseconds.

If not passed or set to 0, it represents an immediate reconnection.

It is recommended to pass a parameter to delay the reconnection to avoid excessive CPU consumption on the local machine due to the inability to connect due to issues with the remote service.

### Return Value
No return value

### Example

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // Reconnect after 1 second if the connection is dropped
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> **Note**
> After a successful reconnection using `reConnect`, the `onConnect` method of `$con` will be called again (if set). Sometimes we may want the `onConnect` method to be executed only once and not upon reconnection. Refer to the following example:

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        static $is_first_connect = true;
        if (!$is_first_connect) return;
        $is_first_connect = false;
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // Reconnect after 1 second if the connection is dropped
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

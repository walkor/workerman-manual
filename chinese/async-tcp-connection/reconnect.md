# reConnect 方法
```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```

 ``` (要求Workerman版本>=3.3.5) ```

重连。一般在```onClose```回调中调用，实现断线重连。

由于网络问题或者对方服务重启等原因导致连接断开，则可以通过调用此方法实现重连。


### 参数
 ``` $delay ```

延迟多久后执行重连。单位为秒，支持小数，可精确到毫秒。

如果不传或者值为0则代表立即重连。

最好传递参数让重连延迟执行，避免因为对端服务问题一直不可连导致本机cpu消耗过高。


### 返回值
无返回值

### 示例

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
        // 如果连接断开，则在1秒后重连
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> **注意**
> 调用reconnect成功重连后，$con的onConnect方法会再次被调用(如果有设置的话)。有时候我们想让onConnect方法只执行一次，重连的时候不再执行，参考如下例子：

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
        // 如果连接断开，则在1秒后重连
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```


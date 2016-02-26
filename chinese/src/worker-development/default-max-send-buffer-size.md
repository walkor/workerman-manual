# defaultMaxSendBufferSize
## 说明:
```php
static int Connection::$defaultMaxSendBufferSize
```

此属性为全局静态属性，用来设置所有连接的默认应用层发送缓冲区大小。不设置默认为```1MB```。 ```Connection::$defaultMaxSendBufferSize```可以动态设置，设置后只对之后产生的新连接有效

此属性影响onBufferFull回调


## 范例

```php
use Workerman\Worker;
use Workerman\Protocols\TcpConnection;
require_once './Workerman/Autoloader.php';

// 设置所有连接的默认应用层发送缓冲区大小
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // 设置当前连接的应用层发送缓冲区大小，会覆盖掉默认值
    $connection->maxSendBufferSize = 4*1024*1024;
};
// 运行worker
Worker::runAll();
```

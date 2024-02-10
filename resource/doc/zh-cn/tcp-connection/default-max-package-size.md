# maxPackageSize

## 说明:
```php
static int Connection::$defaultMaxPackageSize
```

此属性为全局静态属性，用来设置每个连接能够接收的最大包包长。不设置默认为10MB。

如果发来的数据包解析(协议类的input方法返回值)得到包长大于```Connection::$defaultMaxPackageSize```，则会视为非法数据，连接会断开。


## 范例


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 设置每个连接接收的数据包最大为1024000字节
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// 运行worker
Worker::runAll();
```

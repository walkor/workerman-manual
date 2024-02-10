# protocol

## 说明:
```php
string Connection::$protocol
```

设置当前连接的协议类


## 范例


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    $connection->protocol = 'Workerman\\Protocols\\Http';
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    // send 时会自动调用$connection->protocol::encode()，打包数据后再发送
    $connection->send("hello");
};
// 运行worker
Worker::runAll();
```

# connect 方法
```php
void AsyncUdpConnection::connect()
```
执行异步连接操作。此方法会立刻返回。

### 参数
无参数


### 返回值
无返回值

### 示例 

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';


$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1秒后启动一个udp客户端，连接1234端口并发送字符串 hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // 收到服务端返回的数据 hello
            echo "recv $data\r\n";
            // 关闭连接
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // 收到AsyncUdpConnection客户端发来的数据，返回字符串 hello
    $connection->send("hello");
};
Worker::runAll();             
```

执行后打印类似:
```
recv hello
```
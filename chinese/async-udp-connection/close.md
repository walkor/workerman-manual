```php
void Connection::close(mixed $data = '')
```

安全的关闭连接，并触发连接的```onClose```回调。

虽然udp是无连接的，但是对应的AsyncUdpConnection对象却是一直保留在内存中，必须调用close方法才可以释放掉对应的udp连接对象，否则这个udp连接对象会一直存在于内存中，造成内存泄漏。

## 参数

 ``` $data ```

可选参数，要发送的数据（如果有指定协议，则会自动调用协议的encode方法打包```$data```数据），当数据发送完毕后关闭连接，随后会触发onClose回调。

数据大小不能超过65507字节，否则会发送失败。

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


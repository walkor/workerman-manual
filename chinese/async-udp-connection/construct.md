# __construct 方法
```php
void AsyncUdpConnection::__construct(string $remote_address)
```
创建一个udp连接对象。

AsyncUdpConnection可以让Workerman作为客户端与远程服务端传输udp数据。

## 参数
参数:``` remote_address ```

连接的地址，例如
 ``` udp://192.168.1.1:1234 ```
 ``` frame://192.168.1.1:8080 ```
 ``` text://192.168.1.1:8080 ```



## 示例

```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Lib\Timer;

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1秒后启动一个udp客户端，连接1234端口并发送字符串 hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // 收到服务端返回的数据 hello
            echo "recv $data\r\n";
            // 关闭连接
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
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



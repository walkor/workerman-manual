# connect 方法
```php
void AsyncUdpConnection::connect()
```
執行非同步連接操作。此方法會立刻返回。

### 參數
無參數

### 返回值
無返回值

### 範例 
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1秒後啟動一個udp客戶端，連接1234端口並發送字符串 hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // 收到服務端返回的數據 hello
            echo "recv $data\r\n";
            // 關閉連接
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // 收到AsyncUdpConnection客戶端發來的數據，返回字符串 hello
    $connection->send("hello");
};
Worker::runAll();
```
執行後打印類似:
```
recv hello
```

# __construct 方法
```php
void AsyncUdpConnection::__construct(string $remote_address)
```
建立一個UDP連接物件。

AsyncUdpConnection可以讓Workerman作為客戶端與遠端服務器進行UDP數據傳輸。

## 參數
參數:``` remote_address ```

連接的地址，例如
 ``` udp://192.168.1.1:1234 ```
 ``` frame://192.168.1.1:8080 ```
 ``` text://192.168.1.1:8080 ```



## 示例

```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1秒後啟動一個UDP客戶端，連接1234端口並發送字符串 hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // 收到服務器返回的數據 hello
            echo "recv $data\r\n";
            // 關閉連接
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
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

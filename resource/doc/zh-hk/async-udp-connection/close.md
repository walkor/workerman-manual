```php
void Connection::close(mixed $data = '')
```

安全嘅關閉連接，同時觸發連接嘅```onClose```回調。

雖然udp係無連接嘅，但係對應嘅AsyncUdpConnection對象卻係一直保留喺內存中，必須調用close方法先可以釋放對應嘅udp連接對象，唔係嘅話呢個udp連接對象會一直存在喺內存中，造成內存洩漏。

## 參數

 ``` $data ```

可選參數，想要發送嘅數據（如果有指定協議，就會自動調用協議嘅encode方法打包```$data```數據），當數據發送完畢後關閉連接，之後會觸發onClose回調。

數據大小唔可以超過65507字節，否則會發送失敗。

### 示例 

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1秒後啟動一個udp客戶端，連接1234端口同傳送字符串 hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // 收到服務端返回嘅數據 hello
            echo "recv $data\r\n";
            // 關閉連接
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // 收到AsyncUdpConnection客戶端發嚟嘅數據，返還字符串 hello
    $connection->send("hello");
};
Worker::runAll();             
```

執行之後打印類似:
```
recv hello
```

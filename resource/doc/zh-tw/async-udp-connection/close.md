```php
void Connection::close(mixed $data = '')
```

安全地關閉連線，並觸發連線的```onClose```回調。

雖然 UDP 是無連線的，但對應的 AsyncUdpConnection 物件卻一直保留在記憶體中，必須呼叫 close 方法才能釋放對應的 UDP 連接物件，否則這個 UDP 連接物件會一直存在於記憶體中，造成記憶體洩漏。

## 參數

 ``` $data ```

可選參數，要發送的資料（如果有指定協議，則會自動調用協議的 encode 方法打包```$data```資料），當資料發送完畢後關閉連線，隨後會觸發 onClose 回調。

資料大小不能超過65507字節，否則會發送失敗。

### 範例 

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1秒後啟動一個 UDP 客戶端，連接1234端口並發送字符串 hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // 收到服務端返回的資料 hello
            echo "recv $data\r\n";
            // 關閉連線
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // 收到 AsyncUdpConnection 客戶端發來的資料，返回字符串 hello
    $connection->send("hello");
};
Worker::runAll();             
```

執行後打印類似:
```
recv hello
```

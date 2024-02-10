# worker
## 說明:
```php
Worker Connection::$worker
```

這個屬性是一個唯讀屬性，表示當前連線物件所屬的worker實例。

## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// 當客戶端發送數據時，轉發給當前進程所維護的其他所有客戶端
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// 運行worker
Worker::runAll();
```

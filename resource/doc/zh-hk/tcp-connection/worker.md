# worker
## 說明:
```php
Worker Connection::$worker
```

此屬性為唯讀屬性，即當前connection對象所屬的worker實例


## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// 當一個客戶端發來數據時，轉發給當前進程所維護的其他所有客戶端
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

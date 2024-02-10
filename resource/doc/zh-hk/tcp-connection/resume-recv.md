# resumeRecv
## 說明:
```php
void Connection::resumeRecv(void)
```

使當前連線繼續接收數據。這個方法與Connection::pauseRecv配合使用，對於上傳流量控制非常有用

## 參數

無參數

## 範例

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // 給connection對象動態添加一個屬性，用來保存當前連線發來多少個請求
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 每個連接接收100個請求後就不再接收數據
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // 30秒後恢復接收數據
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// 運行worker
Worker::runAll();
```

## 參見
void Connection::pauseRecv(void) 使得對應連接對象停止接收數据

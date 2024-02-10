# maxSendBufferSize
## 說明:
```php
int Connection::$maxSendBufferSize
```

每個連接都有一個獨立的應用層發送緩衝區，如果客戶端接收速度小於服務端發送速度，數據會在應用層緩衝區暫存等待發送。

此屬性用來設置當前連接的應用層發送緩衝區大小。不設置默認為[Connection::defaultMaxSendBufferSize] (default-max-send-buffer-size.md)(1MB)。

此屬性影響[onBufferFull](../worker/on-buffer-full.md)回調。


## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // 設置當前連接的應用層發送緩衝區大小為102400字節
    $connection->maxSendBufferSize = 102400;
};
// 運行worker
Worker::runAll();
```

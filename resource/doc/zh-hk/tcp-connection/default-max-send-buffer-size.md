# defaultMaxSendBufferSize
## 說明:
```php
static int Connection::$defaultMaxSendBufferSize
```

此屬性為全局靜態屬性，用來設置所有連線的默認應用層發送緩衝區大小。如果未設置，則默認為```1MB```。```Connection::$defaultMaxSendBufferSize```可以動態設置，設置後只對之後產生的新連線有效。

此屬性影響[onBufferFull](../worker/on-buffer-full.md)回調。

## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 設置所有連線的默認應用層發送緩衝區大小
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // 設置當前連線的應用層發送緩衝區大小，會覆蓋掉默認值
    $connection->maxSendBufferSize = 4*1024*1024;
};
// 運行worker
Worker::runAll();
```

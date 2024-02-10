# defaultMaxSendBufferSize
## 說明:
```php
static int Connection::$defaultMaxSendBufferSize
```

此屬性為全域靜態屬性，用來設定所有連線的預設應用層發送緩衝區大小。若未設定，預設值為```1MB```。```Connection::$defaultMaxSendBufferSize```可以動態設定，設定後僅對之後產生的新連線有效。

此屬性影響[onBufferFull](../worker/on-buffer-full.md)回調。


## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 設定所有連線的預設應用層發送緩衝區大小
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // 設定當前連線的應用層發送緩衝區大小，將覆蓋掉預設值
    $connection->maxSendBufferSize = 4*1024*1024;
};
// 執行worker
Worker::runAll();
```

# maxPackageSize

## 說明:
```php
static int Connection::$defaultMaxPackageSize
```

此屬性為全域靜態屬性，用來設定每個連線可以接收的最大包長。若未設定，則預設為10MB。

如果收到的資料包解析(協議類的input方法返回值)得到的包長大於```Connection::$defaultMaxPackageSize```，則會視為非法資料，連線將會斷開。

## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 設定每個連線接收的資料包最大為1024000位元組
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// 運行worker
Worker::runAll();
```

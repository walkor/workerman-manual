# maxPackageSize

## 說明:
```php
static int Connection::$defaultMaxPackageSize
```

此屬性為全局靜態屬性，用來設置每個連接能夠接收的最大封包長度。如未設置，預設值為10MB。

如果來自對方的數據封包解析(協議類的input方法返回值)得到的封包長度大於```Connection::$defaultMaxPackageSize```，則會被視為非法數據，並斷開連接。

## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 設置每個連接接收的數據封包最大為1024000位元組
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// 運行worker
Worker::runAll();
```

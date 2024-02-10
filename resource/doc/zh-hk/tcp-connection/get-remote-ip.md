# getRemoteIp
## 說明:
```php
string Connection::getRemoteIp()
```

獲得該連接的客戶端IP

## 參數

無參數

## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "來自IP " . $connection->getRemoteIp() . " 的新連接\n";
};
// 運行worker
Worker::runAll();
```

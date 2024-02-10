# getRemoteIp
## 說明:
```php
string Connection::getRemoteIp()
```

取得該連線的客戶端IP

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
    echo "new connection from ip " . $connection->getRemoteIp() . "\n";
};
// 運行worker
Worker::runAll();
```

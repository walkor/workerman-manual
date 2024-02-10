# getRemotePort
## 說明:
```php
int Connection::getRemotePort()
```

獲取該連接的客戶端端口

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
    echo "new connection from address " .
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
// 運行worker
Worker::runAll();
```

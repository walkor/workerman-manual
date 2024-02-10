# destroy
## 說明:
```php
void Connection::destroy()
```

立即關閉連接。

與close不同之處是，調用destroy後即使該連接的发送緩衝區還有數據未發送到對端，連接也會立即被關閉，並立即觸發該連接的```onClose```回調。

## 參數

無參數

## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // if something wrong
    $connection->destroy();
};
// 運行worker
Worker::runAll();
```

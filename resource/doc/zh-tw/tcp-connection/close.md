# close
## 說明:
```php
void Connection::close(mixed $data = '')
```

安全地關閉連線。

調用close會等待發送緩衝區的數據發送完畢後才關閉連線，並觸發連線的```onClose```回調。

## 參數

 ``` $data ```

可選參數，要發送的數據（如果有指定協議，則會自動調用協議的encode方法打包```$data```數據），當數據發送完畢後關閉連線，隨後會觸發onClose回調

## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// 運行worker
Worker::runAll();
```

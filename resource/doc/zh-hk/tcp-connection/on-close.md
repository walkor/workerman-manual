# onClose
## 說明:
```php
callback Connection::$onClose
```

此回調與[Worker::$onClose](../worker/on-close.md)回調作用相同，區別是只針對當前連接有效，也就是可以針對某個連接的設置onClose回調。

## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 當有連接事件時觸發
$worker->onConnect = function(TcpConnection $connection)
{
    // 設置連接的onClose回調
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "connection closed\n";
    };
};
// 運行worker
Worker::runAll();
```

上面代碼與下面的效果相同

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 設置所有連接的onclose回調
$worker->onClose = function(TcpConnection $connection)
{
    echo "connection closed\n";
};
// 運行worker
Worker::runAll();
```

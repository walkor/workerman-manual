# onClose
## 說明:
```php
callback Connection::$onClose
```

此回撥與[Worker::$onClose](../worker/on-close.md)回撥作用相同，區別是只針對當前連線有效,也就是可以針對某個連線的設置onClose回撥。

## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 當有連線事件時觸發
$worker->onConnect = function(TcpConnection $connection)
{
    // 設置連線的onClose回撥
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "connection closed\n";
    };
};
// 運行worker
Worker::runAll();
```

上面程式碼與下面的效果相同

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 設置所有連線的onclose回撥
$worker->onClose = function(TcpConnection $connection)
{
    echo "connection closed\n";
};
// 運行worker
Worker::runAll();
```

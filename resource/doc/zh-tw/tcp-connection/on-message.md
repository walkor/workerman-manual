# onMessage
## 說明:
```php
callback Connection::$onMessage
```

與[Worker::$onMessage](../worker/on-message.md)回調功能相同，不同之處在於僅對當前連接有效，也就是可以針對某個連接設置onMessage回調。

## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 當有客戶端連接事件時
$worker->onConnect = function(TcpConnection $connection)
{
    // 設置連接的onMessage回調
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('receive success');
    };
};
// 運行worker
Worker::runAll();
```

上面的程式碼與下面的效果是一樣的

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 直接設置所有連接的onMessage回調
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// 運行worker
Worker::runAll();
```

# onClose
## 説明:
```php
callback Connection::$onClose
```

このコールバックは、[Worker::$onClose](../worker/on-close.md)コールバックと同じく、異なるのは現在の接続にのみ有効であり、つまり特定の接続にonCloseコールバックを設定できる点です。

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 接続イベント発生時に実行
$worker->onConnect = function(TcpConnection $connection)
{
    // 接続のonCloseコールバックを設定
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "connection closed\n";
    };
};
// Workerを実行
Worker::runAll();
```

上記のコードは、以下のコードと同等です

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// すべての接続にoncloseコールバックを設定
$worker->onClose = function(TcpConnection $connection)
{
    echo "connection closed\n";
};
// Workerを実行
Worker::runAll();
```

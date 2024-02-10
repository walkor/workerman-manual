# onMessage
## 説明:
```php
callback Connection::$onMessage
```

このコールバックは、[Worker::$onMessage](../worker/on-message.md)コールバックと同じ機能を持っており、異なる点は現在の接続にのみ有効であることです。つまり、特定の接続に対してonMessageコールバックを設定することができます。

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// クライアントが接続したとき
$worker->onConnect = function(TcpConnection $connection)
{
    // 接続のonMessageコールバックを設定する
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('受信成功');
    };
};
// Workerを実行
Worker::runAll();
```

上記のコードと以下のコードは同じ結果をもたらします。

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// すべての接続のonMessageコールバックを直接設定
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('受信成功');
};
// Workerを実行
Worker::runAll();
```

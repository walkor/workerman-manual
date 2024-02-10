# worker
## 説明：
```php
Worker Connection::$worker
```

このプロパティは読み取り専用であり、現在の接続オブジェクトが属するworkerインスタンスです。

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// クライアントからデータを受け取った場合、現在のプロセスで維持している他のすべてのクライアントに転送する
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// workerを実行する
Worker::runAll();
```

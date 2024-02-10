# getRemoteIp
## 説明:
```php
string Connection::getRemoteIp()
```

接続のクライアントIPを取得します。

## パラメーター

パラメーターはありません。

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "IPからの新しい接続：" . $connection->getRemoteIp() . "\n";
};
// Workerを実行
Worker::runAll();
```

# getRemotePort
## 説明:
```php
int Connection::getRemotePort()
```

この接続のクライアントのポートを取得します。

## パラメーター

パラメーターはありません


## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "アドレスからの新しい接続 " .
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
// workerを実行
Worker::runAll();
```

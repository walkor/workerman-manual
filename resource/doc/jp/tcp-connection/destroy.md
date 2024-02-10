# destroy
## 説明:
```php
void Connection::destroy()
```

接続を即座に閉じます。

closeとは異なり、destroyを呼び出すと、接続の送信バッファにまだ送信されていないデータがあっても、接続は即座に閉じられ、接続の```onClose```コールバックが即座にトリガーされます。

## パラメーター

パラメーターなし

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // もし問題があれば
    $connection->destroy();
};
// workerを実行
Worker::runAll();
```

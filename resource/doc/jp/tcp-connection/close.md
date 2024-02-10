# close
## 説明:
```php
void Connection::close(mixed $data = '')
```

安全な接続の終了。

closeを呼び出すと、接続が閉じる前に送信バッファのデータが送信されるのを待ち、接続の```onClose```コールバックがトリガーされます。

## パラメーター

 ```$data```

オプションのパラメーターで、送信するデータ（プロトコルが指定されている場合、```$data```データを自動的にプロトコルのencodeメソッドでパッケージ化します）。データが送信された後に接続が閉じられ、その後```onClose```コールバックがトリガーされます。

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// workerを実行
Worker::runAll();
```

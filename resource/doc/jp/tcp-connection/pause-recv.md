# pauseRecv
## 説明:
```php
void Connection::pauseRecv(void)
```

現在の接続をデータを受信しないように停止します。この接続のonMessageコールバックはトリガされません。このメソッドはトラフィック制御のために非常に有用です。

## パラメーター

パラメーターはありません


## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // connectionオブジェクトにダイナミックにプロパティを追加して、現在の接続から送信されたリクエストの数を保存します
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 各接続が100個のリクエストを受信した後は、データを受信しないようにします
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
    }
};
// workerを実行
Worker::runAll();
```

## 関連
void Connection::resumeRecv(void) - 対応する接続オブジェクトのデータ受信を再開します

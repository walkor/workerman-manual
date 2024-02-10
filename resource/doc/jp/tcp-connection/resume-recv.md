# resumeRecv
## 説明:
```php
void Connection::resumeRecv(void)
```
現在の接続でデータの受信を再開します。このメソッドはConnection::pauseRecvと共に使用し、アップロードトラフィックの制御に非常に役立ちます。

## パラメーター
パラメーターはありません。

## 例
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // 接続オブジェクトに動的にプロパティを追加して、現在の接続からどれだけのリクエストが送信されたかを保存します
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 各接続が100個のリクエストを受信すると、それ以上データの受信を停止します
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // 30秒後にデータ受信を再開します
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// Workerを実行します
Worker::runAll();
```

## 関連項目
void Connection::pauseRecv(void) 接続オブジェクトをデータの受信を停止させます

# connect メソッド
```php
void AsyncUdpConnection::connect()
```
非同期接続操作を実行します。このメソッドは即座にリターンします。

### パラメータ
パラメータはありません。

### 戻り値
戻り値はありません。

### 例

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';


$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1秒後にUDPクライアントを起動し、1234ポートに接続して文字列を送信
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // サーバーからの応答データ hello を受信
            echo "recv $data\r\n";
            // 接続を閉じる
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // AsyncUdpConnectionクライアントからのデータを受け取り、文字列 hello を返す
    $connection->send("hello");
};
Worker::runAll();             
```

実行後、次のように表示されます：
```php
recv hello
```

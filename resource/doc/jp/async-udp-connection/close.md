```php
void Connection::close(mixed $data = '')
```

安全に接続を閉じ、接続の```onClose```コールバックをトリガーします。

UDPは非接続型ですが、対応するAsyncUdpConnectionオブジェクトは常にメモリに保持されており、対応するUDP接続オブジェクトを解放するためにはcloseメソッドを呼び出す必要があります。そうしないと、このUDP接続オブジェクトはメモリ中に永遠に存在し、メモリリークが発生します。

## パラメータ

``` $data ```

オプションのパラメータで、送信するデータ（プロトコルが指定されている場合は、自動的にプロトコルのencodeメソッドが```$data```データをパックします）。データの送信が完了した後、接続が閉じられ、その後にonCloseコールバックがトリガーされます。

データのサイズは65507バイトを超えることはできません。そうでないと送信が失敗します。

### 例

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1秒後にUDPクライアントを起動し、1234ポートに接続して文字列hiを送信します。
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // サーバーからの返信データ hello を受信
            echo "recv $data\r\n";
            // 接続を閉じる
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // AsyncUdpConnectionクライアントからのデータを受信し、文字列helloを返信
    $connection->send("hello");
};
Worker::runAll();             
```

実行後に類似の出力が表示されます:
```php
recv hello
```

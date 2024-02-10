# __construct メソッド
```php
void AsyncUdpConnection::__construct(string $remote_address)
```
UDP接続オブジェクトを作成します。

AsyncUdpConnectionは、Workermanをクライアントとして使用してリモートサーバーとのUDPデータ転送を行うことができます。

## パラメータ
パラメータ：```remote_address```

接続先のアドレス、例えば
 ```udp://192.168.1.1:1234```
 ```frame://192.168.1.1:8080```
 ```text://192.168.1.1:8080```

## 例

```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1秒後にUDPクライアントを起動し、1234ポートに接続して文字列"hi"を送信する
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // サーバーから返されたデータ"hello"を受信
            echo "recv $data\r\n";
            // 接続を閉じる
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
{
    // AsyncUdpConnectionクライアントからのデータを受信し、文字列"hello"を返す
    $connection->send("hello");
};
Worker::runAll();             
```

実行後に以下のような出力が表示されます:
``` 
recv hello
```

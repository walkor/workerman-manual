# onClose
## 説明：
```php
callback Worker::$onClose
```
クライアントの接続がWorkermanから切断されたときにトリガーされるコールバック関数です。接続がどのように切断されたかに関係なく、切断されると```onClose```がトリガーされます。各接続は一度だけ```onClose```をトリガーします。

注意：対向がネットワークの障害や停電などの極端な状況で接続が切断された場合、Workermanは時差すぐにtcpのfinパケットを送信できないため、接続が切断されたことを知らないまま```onClose```をトリガーできません。このような場合はアプリケーションレベルのハートビートで解決する必要があります。Workermanの接続ハートビートの実装については[こちら](https://www.workerman.net/faq/heartbeat.html)を参照してください。GatewayWorkerフレームワークを使用している場合は、GatewayWorkerフレームワークのハートビートメカニズムを直接使用してください。[こちら](https://doc2.workerman.net/heartbeat.html)を参照してください。

UDPは非接続ですので、UDPを使用するとonConnectコールバックがトリガーされず、onCloseコールバックもトリガーされません。

## コールバック関数の引数

```$connection```

接続オブジェクト、つまり[TcpConnectionインスタンス](https://www.workerman.net/tcp-connection)で、クライアント接続の操作に使用されます。例：[データの送信](https://www.workerman.net/tcp-connection/send)，[接続の切断](https://www.workerman.net/tcp-connection/close)など。

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function(TcpConnection $connection)
{
    echo "connection closed\n";
};
// ワーカーを実行
Worker::runAll();
```
注意：コールバックとして無名関数を使用するだけでなく、[こちら](https://www.workerman.net/faq/callback_methods)を参照して他のコールバックの書き方を使用することもできます。

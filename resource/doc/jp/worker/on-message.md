# onMessage
## 説明：
```php
callback Worker::$onMessage
```
クライアントが接続してデータを送信した場合（Workermanがデータを受信した場合）にトリガーされるコールバック関数。

## コールバック関数のパラメータ

```$connection```
接続オブジェクト、つまり[TcpConnectionインスタンス](../tcp-connection.md)、クライアント接続の操作に使用されます。例：[データを送信](../tcp-connection/send.md)、[接続を閉じる](../tcp-connection/close.md)など。

```$data```
クライアント接続から送信されたデータ。Workerがプロトコルを指定している場合、$dataは対応するプロトコルでdecode（解除）されたデータです。データの型はプロトコルの`decode()`実装に依存しており、`websocket` `text` `frame`は文字列で、HTTPプロトコルは[`Workerman\Protocols\Http\Request`](../http/request.md)オブジェクトです。

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// workerを実行
Worker::runAll();
```

ヒント：無名関数をコールバックとして使用するだけでなく、他のコールバックの書き方については[こちらを参照](../faq/callback_methods.md)してください。

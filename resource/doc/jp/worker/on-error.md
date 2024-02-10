# onError
## 説明:
```php
callback Worker::$onError
```

クライアントの接続でエラーが発生した場合にトリガーされます。

現在のエラータイプは次の通りです

1、Connection::sendの呼び出しによるクライアント接続の切断による送信失敗（その後、onCloseコールバックがトリガーされます）``` (code:WORKERMAN_SEND_FAIL msg:client closed) ```

2、onBufferFullをトリガーした後に（送信バッファがいっぱいになった状態で）Connection::sendを呼び出し、かつ送信バッファがまだいっぱいの状態で送信が失敗した場合（onCloseコールバックはトリガーされません）``` (code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package) ```

3、AsyncTcpConnectionの非同期接続が失敗した場合（その後、onCloseコールバックがトリガーされます）``` (code:WORKERMAN_CONNECT_FAIL msg:stream_socket_clientの返すエラーメッセージ) ```

## コールバック関数のパラメータ

``` $connection ```

接続オブジェクト、つまり[TcpConnectionインスタンス](../tcp-connection.md)であり、クライアント接続の操作に使用されます。例えば、[データの送信](../tcp-connection/send.md)、[接続のクローズ](../tcp-connection/close.md)などです。

``` $code ```

エラーコード

``` $msg ```

エラーメッセージ

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "error $code $msg\n";
};
// workerを実行
Worker::runAll();
```

注意: 無名関数をコールバックとして使用する以外にも、[こちら](../faq/callback_methods.md)を参照して他のコールバック方法を使用できます。

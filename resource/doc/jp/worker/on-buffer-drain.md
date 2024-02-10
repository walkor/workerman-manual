# onBufferDrain
## 説明:
```php
callback Worker::$onBufferDrain
```

それぞれの接続にはアプリケーションレベルの送信バッファがあり、バッファサイズは```TcpConnection::$maxSendBufferSize```によって決まります。デフォルト値は1MBであり、手動でサイズを変更することができます。変更した場合はすべての接続に影響します。

このコールバックは、アプリケーションレベルの送信バッファのデータが完全に送信された後にトリガーされます。一般的にはonBufferFullと併用され、例えばonBufferFullでは対向先にデータの送信を停止し、onBufferDrainではデータの書き込みを再開するなどの用途に使用されます。


## コールバック関数のパラメータ

 ``` $connection ```

接続オブジェクト、つまり[TcpConnectionインスタンス](../tcp-connection.md)であり、クライアントの接続を操作するために使用されます。例えば、[データの送信](../tcp-connection/send.md)や[接続のクローズ](../tcp-connection/close.md)などがあります。

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "bufferFull and do not send again\n";
};
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "buffer drain and continue send\n";
};
// workerを実行
Worker::runAll();
```

ヒント：無名関数をコールバックとして使用する他にも、[こちらのリンク](../faq/callback_methods.md)を参照して他のコールバックの書き方を使用することもできます。

## 参照
onBufferFull：接続のアプリケーションレベルの送信バッファが一杯になったときにトリガーされます。

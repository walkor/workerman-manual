# onBufferFull
## 解説:
```php
callback Worker::$onBufferFull
```

それぞれの接続には独自のアプリケーションレベルの送信バッファがあります。クライアント側の受信速度がサーバー側の送信速度よりも遅い場合、データはアプリケーションレベルのバッファに一時的に格納されます。バッファがいっぱいになると、onBufferFullコールバックがトリガーされます。

バッファの最大サイズは[TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md)で設定されており、デフォルト値は1MBです。以下のように、現在の接続に動的にバッファサイズを設定することができます：
```php
// 現在の接続の送信バッファを設定する（単位はバイト）
$connection->maxSendBufferSize = 102400;
```
または、[TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md)を使用して、すべての接続のデフォルトのバッファサイズを設定することもできます：
```php
use Workerman\Connection\TcpConnection;
// すべての接続のデフォルトのアプリケーションレベルの送信バッファのサイズを設定する（単位はバイト）
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```

このコールバックは、Connection::sendの直後にトリガーされる可能性が**あります**。つまり、大量のデータを送信したり、連続してデータを高速で対向端に送信したりすると、ネットワークの問題などにより、対応する接続の送信バッファにデータが大量にたまり、```TcpConnection::$maxSendBufferSize```の上限を超えると発生します。

onBufferFullイベントが発生した場合、通常、送信を停止して対向へのデータ送信を待機したり（onBufferDrainイベントが発生するまで）、開発者は対策を取る必要があります。

Connection::send(`$A`)を呼び出すと、onBufferFullが発生した場合、今回のsendによるデータ`$A`のサイズが大きくても、本当に送信するデータは送信バッファに挿入されます。つまり、実際に送信バッファに挿入されるデータは、通常よりもはるかに大きくなります。送信バッファがすでに```TcpConnection::$maxSendBufferSize```を超えている場合、さらにConnection::send(`$B`)を行うと、今回の`$B`のデータは送信バッファに挿入されず、エラーコールバックがトリガーされます。

要するに、送信バッファがまだいっぱいであっても、1バイトの空きがある限り、Connection::send(```$A```)を呼び出すと必ず```$A```が送信バッファに挿入されます。送信バッファに挿入した後、送信バッファのサイズが```TcpConnection::$maxSendBufferSize```を超えると、onBufferFullコールバックがトリガーされます。

## コールバック関数の引数

 ``` $connection ```

接続オブジェクト、つまり[TcpConnectionインスタンス](../tcp-connection.md)であり、クライアント接続を操作するために使用されます。[データ送信](../tcp-connection/send.md)や[接続を閉じる](../tcp-connection/close.md)などの操作に使用できます。


## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "バッファがいっぱいになりました。送信をもう一度行いません\n";
};
// Workerを実行
Worker::runAll();
```

注意：コールバックとして無名関数を使用する以外にも、[こちら](../faq/callback_methods.md)の他のコールバック書き方を参照してください。

## 関連項目
onBufferDrain 接続のアプリケーションレベルの送信バッファがすべて送信されたときにトリガーされます

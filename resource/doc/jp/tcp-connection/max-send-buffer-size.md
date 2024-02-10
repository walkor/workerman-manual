# maxSendBufferSize
## 説明:
```php
int Connection::$maxSendBufferSize
```

それぞれの接続には、アプリケーションレベルの送信バッファがあります。クライアントの受信速度がサーバーの送信速度よりも遅い場合、データはアプリケーションレベルのバッファに一時的に保持され、送信を待機します。

このプロパティは、現在の接続のアプリケーションレベルの送信バッファのサイズを設定するために使用されます。デフォルトでは設定されていませんが、[Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md)（1MB）がデフォルト値となります。

このプロパティは[onBufferFull](../worker/on-buffer-full.md)コールバックに影響を与えます。

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // 現在の接続のアプリケーションレベルの送信バッファサイズを102400バイトに設定
    $connection->maxSendBufferSize = 102400;
};
// ワーカーを実行する
Worker::runAll();
```

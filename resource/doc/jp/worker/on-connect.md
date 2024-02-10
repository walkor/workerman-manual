# 接続時の動作
## 説明:
```php
callback Worker::$onConnect
```

クライアントがWorkermanと接続したときに（TCPの3-wayハンドシェイクが完了した後）発生するコールバック関数です。各接続は``onConnect``コールバックを1度だけトリガーします。

注意：``onConnect``イベントはクライアントがWorkermanとのTCP接続を完了したことを表すだけであり、この時点ではクライアントからデータが送信されていません。そのため、``$connection->getRemoteIp()``を使用して相手方のIPを取得する以外に、クライアントを識別するためのデータや情報はありません。そのため、相手が誰であるかを知りたい場合は、クライアントが認証データを送信する必要があります。たとえば、トークンやユーザー名とパスワードなどの認証データを[onMessageコールバック](on-message.md)で認証する必要があります。

UDPは無接続なため、UDPを使用している場合、``onConnect``コールバックはトリガーされませんし、``onClose``コールバックもトリガーされません。

## コールバック関数のパラメータ
``` $connection ```

接続オブジェクトで、つまり[TcpConnectionインスタンス](../tcp-connection.md)であり、クライアント接続を操作するためのものです。たとえば、[データを送信する](../tcp-connection/send.md)、[接続を閉じる](../tcp-connection/close.md)などがあります。

## 例
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "IPアドレス" . $connection->getRemoteIp() . "からの新しい接続\n";
};
// Workerの実行
Worker::runAll();
```
ヒント：コールバックとして匿名関数を使用する他にも、[こちら](../faq/callback_methods.md)を参照して別のコールバックの書き方を使用することができます。

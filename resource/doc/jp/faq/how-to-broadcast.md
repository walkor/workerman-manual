# データのブロードキャスト（群送信）方法

## 例（定期的なブロードキャスト）

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// この例ではプロセス数は1でなければなりません
$worker->count = 1;
// プロセスの開始時にタイマーを設定し、定期的にすべてのクライアントにデータを送信します
$worker->onWorkerStart = function($worker)
{
    // 定期的に、10秒ごとに
    Timer::add(10, function()use($worker)
    {
        // 現在のプロセスのすべてのクライアント接続を反復処理し、現在のサーバーの時間を送信します
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// ワーカーを実行
Worker::runAll();
```

## 例（グループチャット）

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// この例ではプロセス数は1でなければなりません
$worker->count = 1;
// クライアントがメッセージを送信したとき、他のユーザーにブロードキャストします
$worker->onMessage = function(TcpConnection $connection, $message)use($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// ワーカーを実行
Worker::runAll();
```

## 説明：
**単一プロセス：**
上記の例は単一プロセス（```$worker->count = 1```）でのみ動作します。複数のプロセスの場合、複数のクライアントが異なるプロセスに接続する可能性があり、プロセス間のクライアントは分離されており、直接通信することはできません。つまり、AプロセスからBプロセスのクライアント接続オブジェクトに直接データを送信することはできません。(これを実現するには、プロセス間通信が必要です。たとえば、Channelコンポーネントを使用することができます。例えば [クラスター送信の例](../components/channel-examples.md)、[グループ送信の例](../components/channel-examples2.md)など)。

**GatewayWorkerの使用をお勧めします**
Workermanをベースに開発されたGatewayWorkerフレームワークは、より便利なプッシュメカニズムを提供しており、グループチャット、ブロードキャストなどを含む、多プロセスおよび複数サーバーの展開が可能です。クライアントにデータをプッシュする必要がある場合、GatewayWorkerフレームワークの使用をお勧めします。

GatewayWorkerマニュアルのURL: https://doc2.workerman.net

# connections
## 説明:
```php
array Worker::$connections
```

形式为
```php
array(id=>connection, id=>connection, ...)
```

この属性には、**現在のプロセス**のすべてのクライアント接続オブジェクトが格納されており、その中でidは接続のid番号であり、詳細についてはマニュアル[TcpConnectionのid属性](../tcp-connection/id.md)を参照してください。

```$connections```は、ブロードキャスト時や接続IDに基づいて接続オブジェクトを取得する際に非常に役立ちます。

もし、接続の番号が```$id```であることが分かっている場合、```$worker->connections[$id]```を使用して対応するconnectionオブジェクトを簡単に取得し、その後ソケット接続を操作することができます。例えば、```$worker->connections[$id]->send('...')```を使用してデータを送信します。

注意：接続が閉じられると（onCloseがトリガーされると）、対応する```connection```は```$connections```配列から削除されます。

注意：開発者はこの属性を変更しないでください。それ以外の場合は、予期しない状況が発生する可能性があります。

## 例

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2020');
// ワーカーの開始時にタイマーを設定し、一定間隔ですべてのクライアント接続にデータを送信する
$worker->onWorkerStart = function($worker)
{
    // タイマー、10秒ごとに
    Timer::add(10, function()use($worker)
    {
        // 現在のプロセスのすべてのクライアント接続を繰り返し、現在のサーバーの時間を送信する
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// ワーカーを実行
Worker::runAll();
```

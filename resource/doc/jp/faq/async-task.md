# 非同期タスクの実装方法

**質問：**

重い業務を非同期で処理し、メイン業務が長時間ブロックされるのを避ける方法は何ですか？例えば、1000人のユーザーにメールを送信する必要がある場合、このプロセスは非常に遅く、数秒かかる可能性があります。このプロセス中にメインプロセスがブロックされるため、後続のリクエストに影響を与えます。このような重いタスクを他のプロセスに非同期に処理させる方法は何ですか。

**答え:**

ローカルマシンや他のサーバー、さらにはサーバークラスターに事前にいくつかのタスクプロセスを立ち上げて、重い業務を処理できます。タスクプロセス数は、例えばCPUの10倍など、多めに起動できます。その後、送信側はAsyncTcpConnectionを使用してこれらのタスクプロセスにデータを非同期に送信し、非同期で処理結果を得ることができます。

タスクプロセスのサーバーサイド
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// タスクワーカー、Textプロトコルを使用
$task_worker = new Worker('Text://0.0.0.0:12345');
// タスクプロセス数は必要に応じて増やすことができます
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function(TcpConnection $connection, $task_data)
{
     // 送られてきたのがJSONデータであると仮定
     $task_data = json_decode($task_data, true);
     // task_dataに基づいて適切なタスクロジックを処理し、結果を得る（ここでは省略）
     $task_result = ......
     // 結果を送信
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

Workermanでの呼び出し方

```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// WebSocketサーバー
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $ws_connection, $message)
{
    // リモートタスクサービスと非同期接続を確立する。IPはリモートタスクサービスのIPであり、ローカルの場合は127.0.0.1、クラスターの場合はLVSのIPになります
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // タスクとパラメータデータ
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // データを送信
    $task_connection->send(json_encode($task_data));
    // 非同期で結果を得る
    $task_connection->onMessage = function(AsyncTcpConnection $task_connection, $task_result)use($ws_connection)
    {
         // 結果
         var_dump($task_result);
         // 結果を取得した後、非同期接続を閉じる
         $task_connection->close();
         // 関連するWebSocketクライアントにタスク完了を通知
         $ws_connection->send('task complete');
    };
    // 非同期接続を実行
    $task_connection->connect();
};

Worker::runAll();
```

このようにすることで、重いタスクをローカルマシンや他のサーバーのプロセスに処理させ、タスクが完了すると非同期で結果を受け取ることができます。メインビジネスプロセスはブロックされずにすみます。

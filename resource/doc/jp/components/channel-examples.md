```php
# 例1
**``` （Workermanのバージョン>=3.3.0が必要）```**

Workerに基づくマルチプロセス（分散クラスター）プッシュシステム、クラスターの送信、クラスターのブロードキャスト。

`start_channel.php`
システム全体で唯一のstart_channelサービスをデプロイできます。192.168.1.1で実行されていると仮定します。
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Channelサーバーの初期化
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```

`start_ws.php`
システム全体で複数のstart_wsサービスをデプロイできます。2台のサーバー、192.168.1.2と192.168.1.3で実行されていると仮定します。
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocketサーバー
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // ChannelクライアントはChannelサーバーに接続する
    Channel\Client::connect('192.168.1.1', 2206);
    // 自分のプロセスIDをイベント名として使用
    $event_name = $worker->id;
    // worker->idのイベントを購読し、イベントハンドラーを登録する
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "接続が存在しません\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // ブロードキャストイベントを購読する
    $event_name = 'ブロードキャスト';
    // ブロードキャストイベントを受信したら、現在のプロセス内のすべてのクライアント接続にブロードキャストデータを送信する
    Channel\Client::on($event_name, function($event_data)use($worker){
        $message = $event_data['content'];
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    });
};

$worker->onConnect = function(TcpConnection $connection)use($worker)
{
    $msg = "workerID:{$worker->id} connectionID:{$connection->id} 接続されました\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```

`start_http.php`
システム全体で複数のstart_wsサービスをデプロイできます。2台のサーバー、192.168.1.4と192.168.1.5で実行されていると仮定します。
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// httpリクエストを処理し、任意のクライアントにデータをプッシュするには、workerIDとconnectionIDを渡す必要があります
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // workerman4.xをサポートする
    if (!is_array($request)) {
            $_GET = $request->get();
   }
    $connection->send('ok');
    if(empty($_GET['content'])) return;
    // 特定のworkerプロセス内の特定の接続にデータをプッシュする
    if(isset($_GET['to_worker_id']) && isset($_GET['to_connection_id']))
    {
        $event_name = $_GET['to_worker_id'];
        $to_connection_id = $_GET['to_connection_id'];
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'to_connection_id' => $to_connection_id,
           'content'          => $content
        ));
    }
    // グローバルブロードキャストデータ
    else
    {
        $event_name = 'ブロードキャスト';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};

Worker::runAll();
```

## テスト
1、各サーバーでサービスを実行します

2、クライアントがサーバーに接続します

Chromeブラウザを開き、F12を押してデバッグコンソールを開き、Consoleタブで以下を入力します（または以下のコードをhtmlページに入れてjavascriptで実行します）

```javascript
// ws://192.168.1.3:4236に接続することもできます
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("サーバーからのメッセージを受信しました：" + e.data);
};
```

3、httpインターフェースを呼び出してプッシュします

urlにアクセスして```http://192.168.1.4:4237/?content={$content}``` または ```http://192.168.1.5:4237/?content={$content}``` で全クライアント接続に```$content```データを送信します

urlにアクセスして```http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` または```http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` で特定のworkerプロセス内の特定のクライアント接続に```$content```データを送信します

注意：テスト時に ```{$worker_id}``` ```{$connection_id}``` そして```{$content}``` を実際の値に置き換えてください
```

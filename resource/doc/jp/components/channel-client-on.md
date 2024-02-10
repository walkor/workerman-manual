**```（要求Workerman版本>=3.3.0）```**
```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```
```$event_name```イベントを購読し、イベント発生時の```$callback_function```コールバックを登録します。

## コールバック関数のパラメータ

 ```$event_name```

購読されているイベントの名称。任意の文字列を使用できます。

 ```$callback_function```

イベント発生時にトリガーされるコールバック関数。関数プロトタイプは```callback_function(mixed $event_data)```です。```$event_data```はイベントの発行時に渡されるイベントデータです。

注意：同じイベントに2つのコールバック関数が登録されている場合、後続のコールバック関数が最初のコールバック関数を上書きします。

## 例
マルチプロセスワーカー（複数のサーバー）で、クライアントがメッセージを送信すると、すべてのクライアントにブロードキャストする。

start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Channelサーバーを初期化
$channel_server = new Channel\Server('0.0.0.0', 2206);

// websocketサーバー
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->name = 'websocket';
$worker->count = 6;
// 各workerプロセスが起動するとき
$worker->onWorkerStart = function($worker)
{
    // ChannelクライアントがChannelサーバーに接続する
    Channel\Client::connect('127.0.0.1', 2206);
    // broadcastイベントを購読し、イベントコールバックを登録
    Channel\Client::on('broadcast', function($event_data)use($worker){
        // 現在のworkerプロセスのすべてのクライアントにメッセージをブロードキャストする
        foreach($worker->connections as $connection)
        {
            $connection->send($event_data);
        }
    });
};

$worker->onMessage = function(TcpConnection $connection, $data)
{
   // クライアントが送信したデータをイベントデータとして扱う
   $event_data = $data;
   // すべてのworkerプロセスにbroadcastイベントを発行する
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```

**テスト**

Chromeブラウザを開き、F12キーを押してデバッグコンソールを開き、Consoleタブで以下のコードを入力します（または以下のコードをHTMLページに配置してJavaScriptで実行します）

メッセージを受信する接続
```javascript
// 127.0.0.1を実際のworkermanのIPに置き換える
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("サーバーからのメッセージを受信：" + e.data);
};
```

メッセージをブロードキャスト
```javascript
ws.send('hello world');
```

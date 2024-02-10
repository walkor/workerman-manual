＃例1
**```（Workermanのバージョン>=3.3.0が必要です）```**

Workerベースの複数プロセスグループ送信システム

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$channel_server = new Channel\Server('0.0.0.0', 2206);

$worker = new Worker('websocket://0.0.0.0:1234');
$worker->count = 8;
// グローバルグループから接続へのマッピング配列
$group_con_map = array();
$worker->onWorkerStart = function(){
    // ChannelクライアントがChannelサーバーに接続
    Channel\Client::connect('127.0.0.1', 2206);

    // グローバルグループ送信メッセージイベントをリッスン
    Channel\Client::on('send_to_group', function($event_data){
        $group_id = $event_data['group_id'];
        $message = $event_data['message'];
        global $group_con_map;
        var_dump(array_keys($group_con_map));
        if (isset($group_con_map[$group_id])) {
            foreach ($group_con_map[$group_id] as $con) {
                $con->send($message);
            }
        }
    });
};
$worker->onMessage = function(TcpConnection $con, $data){
    // グループに参加するメッセージ{"cmd":"add_group", "group_id":"123"}
    // または グループへのメッセージ送信{"cmd":"send_to_group", "group_id":"123", "message":"これはメッセージです"}
    $data = json_decode($data, true);
    var_dump($data);
    $cmd = $data['cmd'];
    $group_id = $data['group_id'];
    switch($cmd) {
        //  接続をグループに追加
        case "add_group":
            global $group_con_map;
            // 接続を対応するグループ配列に追加
            $group_con_map[$group_id][$con->id] = $con;
            // この接続がどのグループに参加したかを記録しておき、onclose時にgroup_con_mapの対応するグループのデータをクリアできるようにする
            $con->group_id = isset($con->group_id) ? $con->group_id : array();
            $con->group_id[$group_id] = $group_id;
            break;
        // グループにメッセージを送信
        case "send_to_group":
            // Channel\Clientが全サーバーの全プロセスに対してグループへのメッセージ送信イベントをブロードキャストする
            Channel\Client::publish('send_to_group', array(
                'group_id'=>$group_id,
                'message'=>$data['message']
            ));
            break;
    }
};
// ここで重要なのは、接続を閉じると、接続をグローバルグループデータから削除し、メモリリークを防ぐことです
$worker->onClose = function(TcpConnection $con){
    global $group_con_map;
    //  接続が参加しているすべてのグループを反復処理し、group_con_mapから対応するデータを削除
    if (isset($con->group_id)) {
        foreach ($con->group_id as $group_id) {
            unset($group_con_map[$group_id][$con->id]);
            if (empty($group_con_map[$group_id])) {
                unset($group_con_map[$group_id]);
            }
        }
    }
};

Worker::runAll();
```

## テスト（すべてを127.0.0.1で実行していると仮定します）
1、サーバーを実行する
```txt
php start.php start
Workerman[del.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.4.2          PHP version:7.1.3
------------------------ WORKERS -------------------------------
user          worker         listen                    processes status
liliang       ChannelServer  frame://0.0.0.0:2206       1         [OK] 
liliang       none           websocket://0.0.0.0:1234   12        [OK] 
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

2、クライアントがサーバーに接続する

Chromeブラウザを開き、F12でデバッグコンソールを開き、Consoleタブに以下を入力します（または以下のコードをhtmlページに配置してJavaScriptで実行してください）

```javascript
// サーバーのipアドレスを127.0.0.1と仮定し、テストする場合は実際のサーバーipに変更してください
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
	ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"これはメッセージです"}');
};
```

# リッスン

```php
void Worker::listen(void)
```

Workerをインスタンス化した後にリッスンを実行するためのメソッドです。

このメソッドは、Workerプロセスの起動後に新しいWorkerインスタンスを動的に作成するために使用されます。これにより、同じプロセスで複数のポートを監視し、さまざまなプロトコルをサポートすることができます。ただし、この方法を使用しても、新しいプロセスを動的に作成するわけではなく、onWorkerStartメソッドをトリガーすることもありません。

例えば、http Workerを起動した後にwebsocket Workerをインスタンス化すると、このプロセスはhttpプロトコルでアクセスでき、かつwebsocketプロトコルでアクセスできるようになります。websocket Workerとhttp Workerは同じプロセス内に存在するため、共通のメモリ変数にアクセスでき、すべてのソケット接続を共有できます。これにより、httpリクエストを受信し、websocketクライアントを操作してクライアントにデータをプッシュするなどの効果が得られます。

**注意:**

PHPバージョンが<=7.0の場合、同じポートのWorkerを複数のサブプロセスでインスタンス化することはサポートされていません。例えば、Aプロセスがポート2016を監視するWorkerを作成した場合、Bプロセスはポート2016を監視するWorkerを作成できません。これを行うと、```Address already in use```エラーが発生します。以下のコードは```実行できません```。

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
// 4つのプロセス
$worker->count = 4;
// 各プロセスが起動後に新しいWorkerをリッスンする
$worker->onWorkerStart = function($worker)
{
    /**
     * 4つのプロセスが起動するときに、2016ポートのWorkerを作成する
     * worker->listen()を実行すると、Address already in useエラーが発生します
     * もし、worker->count=1だとエラーは発生しません
     */
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    // リッスンを実行します。ここでAddress already in useエラーが発生します
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Workerを実行
Worker::runAll();
```

PHPバージョンが>=7.0の場合、Worker->reusePort=trueを設定すると、複数のサブプロセスで同じポートのWorkerを作成できます。以下はその例です：

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4つのプロセス
$worker->count = 4;
// 各プロセスが起動後に新しいWorkerをリッスンする
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    // ポートの再利用を設定すると、同じポートをリッスンするWorkerを作成できます（PHP>=7.0が必要）
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // リッスンを実行します。正常にリッスンされます
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Workerを実行
Worker::runAll();
```

### PHPバックエンド例及びクライアントへの即時メッセージプッシュ

**原理:**

1. WebSocket Workerを確立し、クライアントの長時間接続を維持します。

2. WebSocket Worker内部にtext Workerを確立します。

3. WebSocket Workerとtext Workerは同じプロセス内であり、クライアント接続を簡単に共有できます。

4. 特定の独立したPHPバックエンドシステムはtextプロトコルを使用してtext Workerと通信します。

5. text WorkerはWebSocket接続を操作してデータをプッシュします。

**コードと手順**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 1234ポートをリッスンするworkerコンテナを初期化
$worker = new Worker('websocket://0.0.0.0:1234');

/*
 * ここでプロセス数を必ず1に設定する必要があります
 */
$worker->count = 1;
// Workerプロセスが起動した後に、内部通信ポートを開くためのtext Workerを作成します
$worker->onWorkerStart = function($worker)
{
    // インターナルポートを開くことで、内部システムからデータをプッシュできるようにします。テキストプロトコル形式: テキスト+改行文字
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        // $data配列フォーマット、uidがあり、そのuidのページにデータをプッシュします
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // workermanを使って、uidのページにデータをプッシュします
        $ret = sendMessageByUid($uid, $buffer);
        // プッシュ結果を返します
        $connection->send($ret ? 'ok' : 'fail');
    };
    // ## リッスンを実行を実行 ##
    $inner_text_worker->listen();
};
// uidを接続にマッピングするためのプロパティを追加します
$worker->uidConnections = array();
// クライアントがメッセージを送信した場合のコールバック関数
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // 現在のクライアントが認証済みかどうか、つまりuidが設定されているかどうかを判断します
    if(!isset($connection->uid))
    {
       // 認証されていない場合、最初のパケットをuidとして設定します（デモのために簡単な検証は行っていないため）
       $connection->uid = $data;
       /* uidを接続にマッピングしておくことで、uidを指定して接続を簡単に見つけられるようになり、特定のuidにデータをプッシュすることができます */
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

// クライアントが接続を切断した場合
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // 接続が切断されたとき、マッピング情報も削除します
        unset($worker->uidConnections[$connection->uid]);
    }
};

// 認証済みユーザー全員にデータをプッシュします
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// uidにデータをプッシュします
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// すべてのworkerを実行します
Worker::runAll();
```

バックエンドサービスを起動します
 ```php push.php start -d```

フロントエンドでのメッセージ受信用のJavaScriptコード
```javascript
var ws = new WebSocket('ws://127.0.0.1:1234');
ws.onopen = function(){
    var uid = 'uid1';
    ws.send(uid);
};
ws.onmessage = function(e){
    alert(e.data);
};
```

バックエンドでのメッセージ送信コード
```php
// インターナルプッシュポートにソケット接続を確立します
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
// uidフィールドを含むプッシュデータ
$data = array('uid'=>'uid1', 'percent'=>'88%');
// データを送信しますが、5678ポートはTextプロトコルのポートであり、データの末尾に改行文字を追加する必要があります
fwrite($client, json_encode($data)."\n");
// プッシュ結果を読み取ります
echo fread($client, 8192);
```

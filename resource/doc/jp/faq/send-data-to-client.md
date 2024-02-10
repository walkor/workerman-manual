```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// workerコンテナを初期化し、1234ポートをリッスンします
$worker = new Worker('websocket://workerman.net:1234');
// ====ここでプロセス数は1に設定しなければなりません====
$worker->count = 1;
// uidと接続のマッピングを保存するためのプロパティを追加します（uidはユーザーIDまたはクライアントの一意の識別子です）
$worker->uidConnections = array();
// クライアントからメッセージが届いた時に実行されるコールバック関数
$worker->onMessage = function (TcpConnection $connection, $data) {
    global $worker;
    // 現在のクライアントが既に認証されているかどうかを判断します、つまりuidが設定されているかどうかです
    if (!isset($connection->uid)) {
        // 認証されていない場合、最初のパケットをuidとして扱います（ここではデモのため、本当の検証は行っていません）
        $connection->uid = $data;
        /* uidを接続にマッピングして保存することで、uidで接続を簡単に検索できるようになります。
         * 特定のuidにデータを送信することができます
         */
        $worker->uidConnections[$connection->uid] = $connection;
        return $connection->send('login success, your uid is ' . $connection->uid);
    }
    // その他のロジック、特定のuidへの送信またはグローバルブロードキャスト
    // メッセージのフォーマットがuid:messageの場合は、uidにmessageを送信するものとします
    // uidがallの場合はグローバルブロードキャストです
    list($recv_uid, $message) = explode(':', $data);
    // グローバルブロードキャスト
    if ($recv_uid == 'all') {
        broadcast($message);
    }
    // 特定のuidに送信
    else {
        sendMessageByUid($recv_uid, $message);
    }
};

// クライアントが切断された場合
$worker->onClose = function (TcpConnection $connection) {
    global $worker;
    if (isset($connection->uid)) {
        // 接続が切断されたらマッピングを削除します
        unset($worker->uidConnections[$connection->uid]);
    }
};

// 認証されたすべてのユーザーにデータを送信します
function broadcast($message) {
    global $worker;
    foreach ($worker->uidConnections as $connection) {
        $connection->send($message);
    }
}

// uidにデータを送信します
function sendMessageByUid($uid, $message) {
    global $worker;
    if (isset($worker->uidConnections[$uid])) {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
    }
}

// すべてのworkerを実行します（実際には1つのみが定義されています）
Worker::runAll();
```

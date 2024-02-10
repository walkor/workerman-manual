＃基本フロー
（単純なWebsocketチャットルームサーバーを例に）

#### 1、任意の場所にプロジェクトディレクトリを作成します
SimpleChat/のようなディレクトリに入り、 `composer require workerman/workerman` を実行します

#### 2、`vendor/autoload.php` をインクルード（composerインストール後に生成されます）
start.phpファイルを作成し、 `vendor/autoload.php` をインクルードします
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3、プロトコルの選択
ここでは、Textテキストプロトコルを選択します（WorkerManで独自のテキスト+改行形式のプロトコル）

（現在WorkerManはHTTP、Websocket、Textテキストプロトコルをサポートしています。他のプロトコルが必要な場合は、プロトコルチャプターを参照して独自のプロトコルを開発してください）

#### 4、必要に応じてエントリースクリプトを作成する
以下は簡単なチャットルームのエントリーファイルです。

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// クライアントが接続するとUIDを割り当て、接続を保存し、すべてのクライアントに通知します
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // この接続にUIDを割り当てる
    $connection->uid = ++$global_uid;
}

// クライアントがメッセージを送信すると、すべての人に転送します
function handle_message(TcpConnection $connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] said: $data");
    }
}

// クライアントが切断すると、すべてのクライアントにブロードキャストします
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] logout");
    }
}

// テキストプロトコルのWorkerを2347ポートでリッスンする
$text_worker = new Worker("text://0.0.0.0:2347");

// プロセスを1つのみ起動し、これによりクライアント間でデータを送信できます
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();
```

#### 5、テスト
Textプロトコルはtelnetコマンドを使用してテストできます
```shell
telnet 127.0.0.1 2347
```

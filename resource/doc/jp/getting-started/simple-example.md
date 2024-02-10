# 簡単な開発例

## インストール

**workermanのインストール**
空のディレクトリで以下を実行します
`composer require workerman/workerman`

## サンプル1: HTTPプロトコルを使用してWebサービスを提供する

**start.phpファイルを作成**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// HTTP通信で2345ポートを監視するWorkerを作成
$http_worker = new Worker("http://0.0.0.0:2345");

// 4つのプロセスでサービスを提供
$http_worker->count = 4;

// ブラウザからのデータを受信した場合、"hello world"をブラウザに返信する
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // ブラウザに"hello world"を送信する
    $connection->send('hello world');
};

// Workerを実行する
Worker::runAll();
```

**コマンドラインで実行（Windowsユーザーは[cmdコマンドライン](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn)を使用してください）**
```shell
php start.php start
```

**テスト**

サーバーのIPが127.0.0.1であると仮定

ブラウザで以下のURLにアクセス：http://127.0.0.1:2345

**注：**

1. アクセスできない場合は、[クライアント接続の失敗原因](../faq/client-connect-fail.md)を参照してください。

2. サーバーはHTTPプロトコルを使用し、他のプロトコル（例：WebSocket）を直接使用することはできません。

## サンプル2：WebSocketプロトコルを使用してサービスを提供する

**ws_test.phpファイルを作成**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 注：ここでは前の例とは異なり、WebSocketプロトコルを使用しています
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// 4つのプロセスでサービスを提供
$ws_worker->count = 4;

// クライアントからデータを受信したら、「hello $data」をクライアントに返す
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // クライアントに「hello $data」と送信する
    $connection->send('hello ' . $data);
};

// Workerを実行する
Worker::runAll();
```

**コマンドラインで実行**
```shell
php ws_test.php start
```

**テスト**

Chromeブラウザを開いて、F12を押してデバッグコンソールを開き、コンソールに以下のコードを入力します（または以下のコードをHTMLページに入れてJavaScriptを実行します）：

```javascript
// サーバーのIPが127.0.0.1であると仮定
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("接続成功");
    ws.send('tom');
    alert("サーバーに文字列「tom」を送信しました");
};
ws.onmessage = function(e) {
    alert("サーバーからのメッセージを受信しました：" + e.data);
};
```

**注：**

1. アクセスできない場合は、[マニュアルのよくある質問-接続が失敗する](../faq/client-connect-fail.md)を参照してください。

2. サーバーはWebSocketプロトコルを使用し、他のプロトコル（例：HTTP）を直接使用することはできません。

## サンプル3：直接TCPを使用したデータ転送

**tcp_test.phpを作成**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// アプリケーション層プロトコルを使用しないで2347ポートを監視するWorkerを作成
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// 4つのプロセスでサービスを提供
$tcp_worker->count = 4;

// クライアントからデータを受信したら、「hello $data」をクライアントに返す
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // クライアントに「hello $data」と送信する
    $connection->send('hello ' . $data);
};

// Workerを実行する
Worker::runAll();
```

**コマンドラインで実行**

```shell
php tcp_test.php start
```

**テスト：コマンドラインで実行**
（以下はLinuxコマンドラインの効果であり、Windowsでは異なる場合があります）

```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```

**注：**

1. アクセスできない場合は、[マニュアルのよくある質問-接続が失敗する](../faq/client-connect-fail.md)を参照してください。

2. サーバーはプレーンなTCPプロトコルを使用し、WebSocketやHTTPなどの他のプロトコルとは直接通信できません。

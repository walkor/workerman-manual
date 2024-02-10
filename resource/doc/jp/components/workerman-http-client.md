# workerman/http-client
## 説明
[workerman/http-client](https://github.com/walkor/http-client) は非同期HTTPクライアントコンポーネントです。すべてのリクエストとレスポンスは非同期でブロッキングされず、内部に接続プールが組み込まれており、メッセージリクエストとレスポンスはPSR7仕様に準拠しています。

## インストール：
``` shell
composer require workerman/http-client
```

## サンプル：

**get post requestの使用法**

```php
use Workerman\Worker;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $http = new Workerman\Http\Client();

    $http->get('https://example.com/', function ($response) {
        var_dump($response->getStatusCode());
        echo $response->getBody();
    }, function ($exception) {
        echo $exception;
    });

    $http->post('https://example.com/', ['key1' => 'value1', 'key2' => 'value2'], function ($response) {
        var_dump($response->getStatusCode());
        echo $response->getBody();
    }, function ($exception) {
        echo $exception;
    });

    $http->request('https://example.com/', [
        'method' => 'POST',
        'version' => '1.1',
        'headers' => ['Connection' => 'keep-alive'],
        'data' => ['key1' => 'value1', 'key2' => 'value2'],
        'success' => function ($response) {
            echo $response->getBody();
        },
        'error' => function ($exception) {
            echo $exception;
        }
    ]);
};
Worker::runAll();
```

**ファイルのアップロード**

```php
<?php
use Workerman\Worker;

require_once 'vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $http = new Workerman\Http\Client();
    // ファイルのアップロード
    $multipart = new \Workerman\Psr7\MultipartStream([
        [
            'name' => 'file',
            'contents' => fopen(__FILE__, 'r')
        ],
        [
            'name' => 'json',
            'contents' => json_encode(['a'=>1, 'b'=>2])
        ]
    ]);
    $boundary = $multipart->getBoundary();
    $http->request('http://127.0.0.1:8787', [
        'method' => 'POST',
        'version' => '1.1',
        'headers' => ['Connection' => 'keep-alive', 'Content-Type' => "multipart/form-data; boundary=$boundary"],
        'data' => $multipart,
        'success' => function ($response) {
            echo $response->getBody();
        },
        'error' => function ($exception) {
            echo $exception;
        }
    ]);
};

Worker::runAll();
```

**progressストリームリターン**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Http\Client;
use Workerman\Protocols\Http\Chunk;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
use Workerman\Worker;

$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function (TcpConnection $connection, Request $request) {
    $http = new Client();
    $http->request('https://api.bla.cn/v1/chat/completions', [
        'method' => 'POST',
        'data' => json_encode([
            'model' => 'gpt-3.5-turbo',
            'temperature' => 1,
            'stream' => true,
            'messages' => [['role' => 'user', 'content' => 'hello']],
        ]),
        'headers' => [
            'Content-Type' => 'application/json',
            'Authorization' => 'Bearer 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',
        ],
        'progress' => function($buffer) use ($connection) {
            $connection->send(new Chunk($buffer));
        },
        'success' => function($response) use ($connection) {
            $connection->send(new Chunk(''));
        },
    ]);
    $connection->send(new Response(200, [
        //"Content-Type" => "application/octet-stream",
        "Transfer-Encoding" => "chunked",
    ], ' '));
};
Worker::runAll();
```

## オプション

```php
<?php
require __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
$worker = new Worker();
$worker->onWorkerStart = function(){
    $options = [
        'max_conn_per_addr' => 128, // 1つのドメインで保持できる最大並行接続数
        'keepalive_timeout' => 15,  // 通信がない場合に接続を閉じるタイムアウト
        'connect_timeout'   => 30,  // 接続タイムアウト
        'timeout'           => 30,  // レスポンス待ちタイムアウト
    ];
    $http = new Workerman\Http\Client($options);

    $http->get('http://example.com/', function($response){
        var_dump($response->getStatusCode());
        echo $response->getBody();
    }, function($exception){
        echo $exception;
    });
};
Worker::runAll();
```

## コルーチンの使用法

> **注意**
> コルーチンの使用法には、workerman>=5.0、workerman/http-client>=2.0.0、composer require revolt/event-loop ^1.0.0 が必要です。

```php
use Workerman\Worker;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $http = new Workerman\Http\Client();

    $response = $http->get('https://example.com/');
    var_dump($response->getStatusCode());
    echo $response->getBody();

    $response = $http->post('https://example.com/', ['key1' => 'value1', 'key2' => 'value2']);
    var_dump($response->getStatusCode());
    echo $response->getBody();

    $response = $http->request('https://example.com/', [
        'method' => 'POST',
        'version' => '1.1',
        'headers' => ['Connection' => 'keep-alive'],
        'data' => ['key1' => 'value1', 'key2' => 'value2'],
    ]);
    echo $response->getBody();
};
Worker::runAll();
```

コールバック関数を設定しない場合、クライアントは非同期リクエストの結果を同期的に返し、リクエストプロセスは現在のプロセスをブロックせず、つまり並行してリクエストを処理できます。

## 注意：

1. プロジェクトで最初に `require __DIR__ . '/vendor/autoload.php';` をロードする必要があります。

2. すべての非同期コードは、workermanの起動後の実行環境でのみ実行できます。

3. GatewayWorker、PHPSocket.ioなど、workermanで開発されたすべてのプロジェクトをサポートしています。

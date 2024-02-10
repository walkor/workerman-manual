# Fiberコルーチン
workermanは5.0.0から[Fiberコルーチン](https://www.php.net/manual/zh/language.fibers.php)をサポートしています。

> **注意**
> Fiberの機能を使用するには、PHP>=8.1が必要であり、`composer require revolt/event-loop ^1.0.0`をインストールする必要があります。

### 紹介

FiberはPHPに組み込まれたコルーチンであり、PHPコードを中断してから必要な時にその実行を再開することができます。その最大の利点は、開発者が同期的な方法で非同期でブロッキングしないコードを書くことができるため、コードの保守性が大幅に向上する点です。

### サンプル
以下は、コルーチンと非同期コールバックプログラミングの違いを比較するための例です。HTTPインターフェースを呼び出し、1秒遅延して応答する必要があるという要件があるとします。非同期コールバックおよびコルーチンの書き方は次の通りです。

**非同期コールバックの例**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // HTTPインターフェースを要求する
    $http->get('http://example.com/', function ($response) use ($connection) {
        // 1秒遅延して送信
        Timer::add(1, function() use ($connection, $response) {
            // ブラウザにデータを送信
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**コルーチンの例**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // HTTPインターフェースを呼び出す
    $response = $http->get('http://example.com/');
    // 1秒遅延
    Timer::sleep(1);
    // データを送信
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **注意**
> 上記のコードには、`composer require workerman/http-client ^2.0.0`をインストールする必要があります。

これらの方法はどちらも非同期でブロッキングされずに実行され、効率的に動作しますが、コルーチンの使用方法は非同期コールバックよりも読みやすく、メンテナンスしやすいです。

### Fiberの注意点
* FiberコルーチンではPdo、Redis、およびPHPの内部ブロッキング関数をコルーチン化することはサポートされていません。つまり、これらの拡張機能や関数を使用してもブロッキング呼び出しとなります。
* 利用可能なFiberコルーチンクライアントは、[workerman/http-client](../components/workerman-http-client.md)、[workerman/redis](../components/workerman-redis.md)です。

# Swooleコルーチン
workerman v5は、Swooleをベースとしたイベント駆動としてサポートしています。

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// ここでSwooleをイベント駆動のベースとして設定する必要があります
Worker::$eventLoopClass = Workerman\Events\Swoole::class;
$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    Coroutine::create(function() use ($connection) {
        $cli = new Client('example.com', 80);
        $cli->get('/get');
        $connection->send($cli->body);
    });
};

Worker::runAll();
```

**ヒント**
* swoole5.0またはそれ以降のバージョンを使用することをお勧めします。
* swooleをイベント駆動のベースとして設定することで、workermanがswooleのコルーチンをサポートできます。
* swooleをイベント駆動のベースとして使用する場合、event拡張機能をインストールする必要はありません。
* swooleではデフォルトで一括コルーチンは有効になっていないため、Pdo、Redis、PHPの組み込みファイル読み書きはブロッキング呼び出しとなります。
* 一括コルーチンを有効にするには、`\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);` を手動で呼び出す必要があります。

詳細については[Swooleマニュアル](https://wiki.swoole.com/)を参照してください。

詳細については[イベント駆動](appendices/event.md)を参照してください。

# コルーチンについて
まず第一に、コルーチンを過度に迷信する必要はありません。データベースやRedisなどのストレージがすべて内部ネットワークにある場合、多くの場合、多プロセスのブロッキング呼び出しの方がコルーチンよりも高速であることがあります。[techempower.comの3年間のベンチマークデータ](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db)によると、workermanのブロッキング式データベース呼び出しのパフォーマンスは、swooleのデータベース接続プール+コルーチンや、さらにはgo言語のgin、echoなどのコルーチンフレームワークよりもほぼ倍の高性能であることがわかります。

workermanはすでにPHPアプリケーションのパフォーマンスを数倍から数十倍に向上させており、ほとんどのworkermanプロジェクトにコルーチンを追加しても、さらなるパフォーマンス向上は期待できません。
外部HTTPリクエストなどの遅い呼び出しがシステムにある場合は、パフォーマンスを向上させるためにコルーチンを使用することを検討してください。

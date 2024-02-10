# reusePort
> **注意**
> This feature requires workerman >= 3.2.1 and PHP >= 7.0. It is not supported on Windows and Mac OS.

## 説明:

```php
bool Worker::$reusePort
```

現在のWorkerがポートの再利用（ソケットのSO_REUSEPORTオプション）を有効にするかどうかを設定します。

ポートの再利用を有効にすると、関係のない複数のプロセスが同じポートを監視でき、システムカーネルによってソケット接続がどのプロセスに処理されるかを負荷分散し、群れの効果を避け、マルチプロセスの短い接続アプリケーションの性能を向上させることができます。

**注意：** この機能にはPHPのバージョンが7.0以上が必要です。

## 例1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// Workerを実行
Worker::runAll();
```

## 例2：Workermanの多ポート（多プロトコル）リスニング

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// 各プロセスの開始後にカレントプロセスでリスニングを生成
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * 複数のプロセスが同じポートを監視する（親プロセスから継承したリッスンソケットではない）
     * ポートの再利用を有効にする必要があり、そうしない場合は「Address already in use」エラーが発生します
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // リッスンを実行
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

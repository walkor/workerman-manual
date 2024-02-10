# stopAll
```php
void Worker::stopAll(void)
```

現在のプロセスを停止し、終了します。

> **注意**
> `Worker::stopAll()`は現在のプロセスを停止し、現在のプロセスが終了した後、メインプロセスがすぐに新しいプロセスを起動します。ワーカーマン全体のサービスを停止する場合は、`posix_kill(posix_getppid(), SIGINT)` を呼び出してください。

### パラメータ
パラメータはありません。



### 戻り値
戻り値はありません。

## 例 max_request

以下の例では、サブプロセスは1000個のリクエストを処理した後にstopAllを実行して終了し、新しいプロセスを再起動します。これは、php-fpmのmax_request属性に似ており、主にphpのビジネスコードによるメモリリークの問題を解決するために使用されます。

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 各プロセスで最大1000個のリクエストを実行
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // 処理されたリクエスト数
    static $request_count = 0;

    $connection->send('hello http');
    // リクエスト数が1000に達したら
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * 現在のプロセスを終了し、メインプロセスが直ちに新しいプロセスを起動して補充し、
         * プロセスの再起動を完了します。
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```

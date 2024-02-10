# デバッグの基本

WorkerManには、デバッグモードとデーモン実行モードの2つの実行モードがあります。

```php start.php start``` を実行すると、デバッグモードに入ります。このとき、コード内の```echo、var_dump、var_export```などの関数による出力は端末に表示されます。なお、```php start.php start``` で実行されたWorkerManは、端末を閉じると全てのプロセスが終了します。

一方、```php start.php start -d``` を実行すると、デーモンモードに入ります。これは本番での実行モードであり、端末を閉じても影響を受けません。

デーモンモードで```echo、var_dump、var_export```などの関数による出力を見たい場合は、Worker::$stdoutFileプロパティを設定することができます。例えば、次のように設定します。

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 画面の出力をWorker::$stdoutFileで指定したファイルに書き込む
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```

これにより、すべての```echo、var_dump、var_export```などの関数による出力が、```Worker::$stdoutFile```で指定されたファイルに書き込まれます。なお、```Worker::$stdoutFile```で指定されたパスには書き込み権限が必要です。

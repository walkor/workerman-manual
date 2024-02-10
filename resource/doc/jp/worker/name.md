# 名前

## 説明:
```php
string Worker::$name
```

現在のWorkerインスタンスの名前を設定し、プロセスを識別するためにstatusコマンドを実行する際に役立ちます。未設定の場合はデフォルトでnoneになります。

## 例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// インスタンスの名前を設定
$worker->name = 'MyWebsocketWorker';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// workerを実行
Worker::runAll();
```

# カウント

## 説明:
```php
int Worker::$count
```

現在のWorkerインスタンスで起動するプロセス数を設定します。デフォルトでは1に設定されています。

プロセス数の設定方法については[**こちら**](../faq/processes-count.md)を参照してください。

注意：このプロパティは```Worker::runAll();```が実行される前に設定する必要があります。Windowsシステムではこの機能はサポートされていません。

## 例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 8つのプロセスを起動し、同時に8484ポートを監視し、websocketプロトコルでサービスを提供します
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// Workerを実行
Worker::runAll();
```

# onWorkerStart
## 説明：
```php
callback Worker::$onWorkerStart
```

Workerのサブプロセスが起動するときに実行されるコールバック関数を設定します。各サブプロセスの起動時に実行されます。

注意：onWorkerStartはサブプロセスが起動するときに実行されます。複数のサブプロセスが起動している場合（```$worker->count > 1```）、それぞれのサブプロセスが1回ずつ実行され、合計```$worker->count```回実行されます。

## コールバック関数のパラメータ
```$worker```
Workerオブジェクトそのもの

## 例
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "Worker starting...\n";
};
// Workerを起動する
Worker::runAll();
```

ヒント：無名関数をコールバックとして使用する以外に、[こちら](../faq/callback_methods.md)で他のコールバックの書き方も参照できます。

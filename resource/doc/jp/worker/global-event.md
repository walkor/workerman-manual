# globalEvent

## 説明：
```php
static Event Worker::$globalEvent
```

このプロパティはグローバルな静的プロパティであり、グローバルなeventloopインスタンスであり、ファイルディスクリプタの読み書きイベントやシグナルイベントを登録することができます。

## 例

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'Pid is ' . posix_getpid() . "\n";
    // プロセスがSIGALRMシグナルを受信すると、いくつかの情報を出力します
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Get signal SIGALRM\n";
    });
};
// workerを実行する
Worker::runAll();
```

## テスト
Workermanが起動すると、現在のプロセスのpid（数字）が出力されます。コマンドラインで実行する
```bash
kill -SIGALRM プロセスのpid
```
サーバーは以下の出力を行います
```bash
Get signal SIGALRM
```

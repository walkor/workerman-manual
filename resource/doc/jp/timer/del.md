# del
```php
boolean \Workerman\Timer::del(int $timer_id)
```
タイマーを削除する

### パラメーター
``` timer_id ```

タイマーのID、つまりaddインターフェースが返す整数

### 戻り値
boolean

### 例
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// 定时任务を実行するために複数のプロセスを起動します。複数のプロセスで並行処理に注意してください。
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // 2秒ごとに実行
    $timer_id = Timer::add(2, function()
    {
        echo "task run\n";
    });
    // 20秒後に一回だけのタスクを実行し、2秒ごとのタイマータスクを削除します。
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// workerを実行
Worker::runAll();
```

### インスタンス（タイマーコールバックで現在のタイマーを削除する）
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // 注意：コールバック内で現在のタイマーIDを使用する場合は、参照(&)を使用して導入する必要があります。
    $timer_id = Timer::add(1, function()use(&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // 10回実行したらタイマーを削除します。
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// workerを実行
Worker::runAll();
```

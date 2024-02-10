# globalEvent

## 說明:
```php
static Event Worker::$globalEvent
```

此屬性為全局靜態屬性，為全局的eventloop實例，可以向其註冊檔描述符的讀寫事件或者信號事件。

## 範例

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'Pid is ' . posix_getpid() . "\n";
    // 當進程收到SIGALRM信號時，打印輸出一些信息
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Get signal SIGALRM\n";
    });
};
// 運行worker
Worker::runAll();
```

## 測試
Workerman啟動後會輸出當前進程pid(一個數字)。命令行運行
```
kill -SIGALRM 進程pid
```
伺服器會打印出
```
Get signal SIGALRM
```

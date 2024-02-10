# id
要求```（workerman >= 3.2.1）```

## 說明:
```php
int Worker::$id
```

當前worker進程的id編號，範圍從```0```到```$worker->count-1```。

這個屬性對於區分worker進程非常有用，例如1個worker實例有多個進程，開發者只想在其中一個進程中設定定時器，則可以通過識別進程編號id來做到這一點，比如只在該worker實例id編號為0的進程設定定時器（見範例）。

## 注意：

進程重啟後id編號值是不變的。

進程編號id的分配是基於每個worker實例的。每個worker實例都從0開始給自己的進程編號，所以worker實例間進程編號會有重複，但是一個worker實例中的進程編號不會重複。例如以下的例子：

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// worker實例1有4個進程，進程id編號將分別為0、1、2、3
$worker1 = new Worker('tcp://0.0.0.0:8585');
// 設置啟動4個進程
$worker1->count = 4;
// 每個進程啟動後打印當前進程id編號即 $worker1->id
$worker1->onWorkerStart = function($worker1)
{
    echo "worker1->id={$worker1->id}\n";
};

// worker實例2有兩個進程，進程id編號將分別為0、1
$worker2 = new Worker('tcp://0.0.0.0:8686');
// 設置啟動2個進程
$worker2->count = 2;
// 每個進程啟動後打印當前進程id編號即 $worker2->id
$worker2->onWorkerStart = function($worker2)
{
    echo "worker2->id={$worker2->id}\n";
};

// 運行worker
Worker::runAll();
```
輸出類似
```
worker1->id=0
worker1->id=1
worker1->id=2
worker1->id=3
worker2->id=0
worker2->id=1
```
注意：Windows系統由於不支持進程數count的設置，只有id只有一個0號。Windows系統下不支持同一個文件初始化兩個Worker監聽，所以Windows系統這個示例無法運行。


## 範例
一個worker實例有4個進程，只在id編號為0的進程上設置定時器。

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8585');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // 只在id編號為0的進程上設置定時器，其它1、2、3號進程不設置定時器
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4個worker進程，只在0號進程設置定時器\n";
        });
    }
};
// 運行worker
Worker::runAll();
```

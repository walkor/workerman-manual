# workerman/crontab

# 说明
`workerman/crontab` 是一个基于workerman的定时任务程序，类似linux的crontab。`workerman/crontab`支持秒级别定时。

>使用 `workerman/crontab`需要先设置好php的时区，否则运行结果可能和预期的不一致。

## 时间说明
```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[可省略，如果没有0位,则最小时间粒度是分钟]
```

# 安装
```
composer require workerman/crontab
```

# 示例
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// 设置时区，避免运行结果与预期不一致
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // 每分钟的第1秒执行.
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // 每天的7点50执行，注意这里省略了秒位.
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```

> 注意
> 定时任务不会马上执行，所有定时任务进入下一分钟才会开始计时执行。
> 与workerman定时器一样，任务全部在当前进程执行。
> 没有开启swoole/swow协程的情况下，如果任务没执行完到了下个执行周期则会等待当前任务执行完，新任务执行会延后。
> 如果开启了swoole/swow协程，前一个任务如果没有执行完，下一个任务可能会立刻执行，不会排队等待。

# 接口
**Crontab::destroy()**

销毁定时器
```
$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();
```

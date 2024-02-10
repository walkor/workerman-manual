# workerman/crontab

# 說明
`workerman/crontab` 是一個基於workerman的定時任務程序，類似linux的crontab。`workerman/crontab`支持秒級定時。

>使用 `workerman/crontab`需要先設定好php的時區，否則運行結果可能和預期的不一致。

## 時間說明
```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ 週幾 (0 - 6) (星期日=0)
|   |   |   |   +------ 月份 (1 - 12)
|   |   |   +-------- 月份中的某一天 (1 - 31)
|   |   +---------- 小時 (0 - 23)
|   +------------ 分鐘 (0 - 59)
+-------------- 秒 (0-59)[可省略，如果沒有0位,則最小時間粒度是分鐘]
```

# 安裝
```composer require workerman/crontab```


# 範例
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// 設定時區，避免運行結果與預期不一致
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // 每分鐘的第1秒執行.
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // 每天的7點50分執行，注意這裡省略了秒位.
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```
> 注意：定時任務不會馬上執行，所有定時任務進入下一分鐘才會開始計時執行。

# 接口
**Crontab::destroy()**

銷毀定時器
```php
$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();
```

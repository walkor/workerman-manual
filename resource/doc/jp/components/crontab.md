# workerman/crontab

# 説明
`workerman/crontab` は、workermanをベースにしたcronタスクプログラムであり、Linuxのcrontabに似ています。`workerman/crontab` は秒単位のスケジュールをサポートしています。

>`workerman/crontab`を使用するには、まずPHPのタイムゾーンを設定する必要があります。そうしないと、実行結果が予期と異なる場合があります。

## 時間の説明
```plaintext
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ 曜日 (0 - 6) (日曜日=0)
|   |   |   |   +------ 月 (1 - 12)
|   |   |   +-------- 月の日 (1 - 31)
|   |   +---------- 時 (0 - 23)
|   +------------ 分 (0 - 59)
+-------------- 秒 (0-59)[省略可能、0位がない場合、最小単位は分]
```

# インストール
```plaintext
composer require workerman/crontab
```

# 例
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// タイムゾーンを設定して、実行結果が予期しないものにならないようにする
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // 1分毎に1秒目に実行.
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // 毎日の7時50分に実行、ここでは秒の位置を省略していることに注意.
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```

> 注意：タイマータスクはすぐに実行されません。すべてのタイマータスクは次の分に入ってから実行されます。

# インターフェース
**Crontab::destroy()**

タイマーを破棄する
```php
$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();
```

# onWorkerStart
## 說明:
```php
callback Worker::$onWorkerStart
```

設置Worker子進程啟動時的回調函數，每個子進程啟動時都會執行。

注意：onWorkerStart是在子進程啟動時運行的，如果開啟了多個子進程(```$worker->count > 1```)，每個子進程運行一次，則總共會運行```$worker->count```次。


## 回調函數的參數

 ``` $worker ```

即Worker對象



## 範例


```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "Worker starting...\n";
};
// 運行worker
Worker::runAll();
```

提示：除了使用匿名函數作為回調，還可以[參考這裡](../faq/callback_methods.md)使用其他回調寫法。

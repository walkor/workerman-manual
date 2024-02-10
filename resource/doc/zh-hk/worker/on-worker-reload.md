# onWorkerReload
要求```（workerman >= 3.2.5）```
## 說明:
```php
callback Worker::$onWorkerReload
```
這個功能很少用到。

設置Worker收到reload信號後執行的回調。

可以利用onWorkerReload回調做很多事情，例如在不需要重啟進程的情況下重新加載業務配置文件。

**注意**：

子進程收到reload信號默認的動作是退出重啟，以便新進程重新加載業務代碼完成代碼更新。所以reload後子進程在執行完onWorkerReload回調後便立刻退出是正常現象。

如果在收到reload信號後只想讓子進程執行onWorkerReload，不想退出，可以在初始化Worker實例時設置對應的Worker實例的reloadable屬性為false。


## 回調函數的參數

 ``` $worker ```

即Worker對象



## 範例


```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 設置reloadable為false，即子進程收到reload信號不執行重啟
$worker->reloadable = false;
// 執行reload後告訴所有客戶端服務端執行了reload
$worker->onWorkerReload = function(Worker $worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// 運行worker
Worker::runAll();
```

提示：除了使用匿名函數作為回調，還可以[參考這裡](../faq/callback_methods.md)使用其他回調寫法。

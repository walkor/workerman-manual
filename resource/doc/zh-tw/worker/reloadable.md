# reloadable
## 說明:
```php
bool Worker::$reloadable
```

執行`php start.php reload`時會向所有子進程發送reload信號(SIGUSR1)。

子進程收到reload信號後會自動退出，然後主進程會自動啟動一個新的進程，一般用於更新業務程式碼。

當進程$reloadable為false時，收到reload信號後只會觸發 [onWorkerReload](on-worker-reload.md)，並不會重啟當前進程。

例如Gateway/Worker模型中的gateway進程負責維護客戶端連線工作，worker進程負責處理請求。
設定gateway進程的reloadable屬性為false則在reload可以做到在不斷開客戶端連線的情況下更新業務程式碼。


## 範例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 設定此實例收到reload信號後是否重啟
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// 執行worker
Worker::runAll();
```

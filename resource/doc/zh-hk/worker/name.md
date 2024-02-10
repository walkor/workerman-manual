# 名稱

## 說明:
```php
string Worker::$name
```

設置當前Worker實例的名稱，方便運行status命令時識別進程。不設置時默認為none。


## 範例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 設置實例的名稱
$worker->name = 'MyWebsocketWorker';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// 運行worker
Worker::runAll();
```

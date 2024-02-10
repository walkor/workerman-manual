# 守護進程化
## 說明:
```php
static bool Worker::$daemonize
```
這個屬性是一個全域靜態屬性，表示是否以守護進程方式運行。如果啟動命令使用了```-d```參數，則該屬性會自動設置為true。也可以在程式碼中手動設置。

注意：這個屬性必須在```Worker::runAll();```運行前設置才有效。Windows系統不支持這個特性。

## 範例
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// 運行worker
Worker::runAll();
```

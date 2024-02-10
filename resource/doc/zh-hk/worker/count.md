# count

## 說明：
```php
int Worker::$count
```

設定當前 Worker 實例啟動多少個進程，如果不設定，默認為1。

如何設定進程數，請參考[**這裡**](../faq/processes-count.md)。

注意：此屬性必須在```Worker::runAll();```運行前設置才有效。Windows系統不支持此特性。

## 範例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 啟動8個進程，同時監聽8484端口，以websocket協議提供服務
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// 運行worker
Worker::runAll();
```

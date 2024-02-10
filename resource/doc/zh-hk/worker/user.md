# user

## 說明:
```php
string Worker::$user
```

設置當前Worker實例以哪個用戶運行。此屬性只有當前用戶為root時才能生效。不設置時默認以當前用戶運行。

建議```$user```設置權限較低的用戶，例如www-data、apache、nobody等。

注意：此屬性必須在```Worker::runAll();```運行前設置才有效。Windows系統不支持此特性。


## 範例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 設置實例的運行用戶
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// 運行worker
Worker::runAll();
```

# user

## 說明:
```php
string Worker::$user
```

設定當前 Worker 實例以哪個使用者執行。此屬性只有當前使用者為 root 時才能生效。不設定時預設以當前使用者執行。

建議```$user```設定為權限較低的使用者，例如 www-data、apache、nobody 等。

注意：此屬性必須在```Worker::runAll();```執行前設定才有效。Windows 系統不支持此特性。

## 範例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 設定實例的執行使用者
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// 執行 worker
Worker::runAll();
```

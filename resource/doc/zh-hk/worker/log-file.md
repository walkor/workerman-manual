# logFile
## 說明:
```php
static string Worker::$logFile
```

用來指定workerman日誌文件位置。

此文件記錄了workerman自身相關的日誌，包括啟動、停止等。

如果沒有設定，文件名默認為workerman.log，文件位置位於Workerman的上一級目錄中。

**注意：**

這個日誌文件中僅僅記錄workerman自身相關啟動停止等日誌，不包含任何業務日誌。

業務日誌類可以利用[file_put_contents](https://php.net/manual/zh/function.file-put-contents.php) 或者 [error_log](https://php.net/manual/zh/function.error-log.php) 等函數自行實現。

## 範例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// 執行 worker
Worker::runAll();
```

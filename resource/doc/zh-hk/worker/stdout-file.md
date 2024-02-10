# stdoutFile
## 說明:
```php
static string Worker::$stdoutFile
```

此屬性為全域靜態屬性，如果以守護進程方式(```-d```啟動)運行，則所有輸出到終端的輸出(echo var_dump等)都會被重定向到stdoutFile指定的文件中。

如果不設置，並且以守護進程方式運行，則所有終端輸出全部重定向到`/dev/null` (亦即預設捨棄所有輸出)

> 注意：`/dev/null` 是linux下一個特殊文件，它實際上是一個黑洞，所有數據寫入到這個文件都會被丟棄。如果不想丟棄輸出，可以將`Worker::$stdoutFile`設置成一個正常文件路徑。

> 注意：此屬性必須在```Worker::runAll();```運行前設置才有效。windows系統不支援此特性。

## 範例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// 所有的打印輸出全部保存在/tmp/stdout.log文件中
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// 運行worker
Worker::runAll();
```

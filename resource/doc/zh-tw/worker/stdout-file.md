# stdoutFile
## 說明:
```php
static string Worker::$stdoutFile
```

此屬性是全域靜態屬性，如果以守護進程方式(```-d```啟動)運行，則所有輸出至終端的內容(echo var_dump等)都將被重新導向至stdoutFile指定的檔案中。

如果未設置，並且是以守護進程方式運行，則所有終端輸出都會被重新導向到 `/dev/null` (也就是預設丟棄所有輸出)

> 注意：`/dev/null` 是 Linux 下的一個特殊檔案，實際上是一個黑洞，所有寫入到這個檔案的資料都會被丟棄。如果不想丟棄輸出，可以將`Worker::$stdoutFile`設置成一個正常的檔案路徑。

> 注意：此屬性必須在```Worker::runAll();```運行前設定才有效。Windows 系統不支援此特性。

## 範例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// 所有的打印輸出全部保存在/tmp/stdout.log檔案中
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// 運行worker
Worker::runAll();
```

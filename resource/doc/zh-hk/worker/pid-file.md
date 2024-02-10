# pidFile
## 說明:
```php
static string Worker::$pidFile
```
建議不設置此屬性，除非有特殊需要。

此屬性為全局靜態屬性，用於設置WorkerMan進程的pid文件路徑。

在監控中，設置此項比較有用，例如將WorkerMan的pid文件放入固定的目錄中，可以方便一些監控軟件讀取pid文件，從而監控WorkerMan進程狀態。

如果不設置，WorkerMan會在與Workerman目錄平行的位置（注意workerman3.2.3之前版本默認在```sys_get_temp_dir()```下）自動生成一個pid文件，並且為了避免啟動多個WorkerMan實例導致pid衝突，WorkerMan生成pid文件包含了當前WorkerMan的路徑。

注意：此屬性必須在```Worker::runAll();```運行前設置才有效。Windows系統不支持此功能。

## 範例
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$pidFile = '/var/run/workerman.pid';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// 運行worker
Worker::runAll();
```

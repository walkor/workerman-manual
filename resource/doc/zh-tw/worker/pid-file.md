# pidFile
## 說明:
```php
static string Worker::$pidFile
```

若無特殊需要，建議不要設置此屬性。

此為全域靜態屬性，用於設定Workerman進程的pid檔案路徑。

在監控中，此設置非常有用，例如將Workerman的pid檔案放入固定的目錄中，可以方便一些監控軟體讀取pid檔案，進而監控Workerman進程狀態。

若未設置，Workerman預設會在與Workerman目錄平行的位置（注意workerman3.2.3之前版本預設在```sys_get_temp_dir()```下）自動產生一個pid檔案，為了避免啟動多個Workerman實例導致pid衝突，Workerman生成的pid檔案包含了當前Workerman的路徑。

注意：此屬性必須在```Worker::runAll();```運行前設定才有效。不支援在Windows系統下此功能。


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

# logFile
## 说明:
```php
static string Worker::$logFile
```

用来指定workerman日志文件位置。

此文件记录了workerman自身相关的日志，包括启动、停止等。

如果没有设置，文件名默认为workerman.log，文件位置位于Workerman的上一级目录中。

**注意：**

这个日志文件中仅仅记录workerman自身相关的日志，不包含任何业务日志。

## 范例

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// 运行worker
Worker::runAll();
```

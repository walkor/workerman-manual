# stdoutFile
## 说明:
```php
static string Worker::$stdoutFile
```

此属性为全局静态属性，如果以守护进程方式(```-d```启动)运行，则所有向终端的输出(echo var_dump等)都会被重定向到stdoutFile指定的文件中。

如果不设置，并且是以守护进程方式运行，则所有终端输出全部重定向到`/dev/null` (也就是默认丢弃所有输出)

> 注意：`/dev/null` 是linux下一个特殊文件，它实际上是一个黑洞，所有数据写入到这个文件都会被丢弃。如果不想丢弃输出，可以将`Worker::$stdoutFile`设置成一个正常文件路径。

> 注意：此属性必须在```Worker::runAll();```运行前设置才有效。windows系统不支持此特性。

## 范例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// 所有的打印输出全部保存在/tmp/stdout.log文件中
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// 运行worker
Worker::runAll();
```

# daemonize
## 说明:
```php
static bool Worker::$daemonize
```

此属性为全局静态属性，表示是否以daemon(守护进程)方式运行。如果启动命令使用了 ```-d```参数，则该属性会自动设置为true。也可以代码中手动设置。

注意：此属性必须在```Worker::runAll();```运行前设置才有效。windows系统不支持此特性。

## 范例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// 运行worker
Worker::runAll();
```

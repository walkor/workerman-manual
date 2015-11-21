# reloadable
## 说明:
```php
bool Worker::$reloadable
```

设置当前Worker实例是否可以reload，即收到reload信号后是否退出重启。不设置默认为true，收到reload信号后自动重启进程。

有些进程维持着客户端连接，例如Gateway/Worker模型中的gateway进程，当运行reload重新载入业务代码时，却又不想客户端连接断开，则设置gateway进程的reloadable属性为false


## 范例

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 设置此实例收到reload信号后是否reload重启
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// 运行worker
Worker::runAll();
```

# reloadable
## 说明:
```php
bool Worker::$reloadable
```

执行`php start.php reload`时会向所有子进程发送reload信号(SIGUSR1)。

子进程收到reload信号后会自动退出然后主进程会自动拉起一个新的进程，一般用于更新业务代码。

当进程$reloadable为false时，收到reload信号后只会触发 [onWorkerReload](on-worker-reload.md) , 并不会重启当前进程。

例如Gateway/Worker模型中的gateway进程负责维持客户端连接工作，worker进程负责处理请求。
设置gateway进程的reloadable属性为false则在reload可以做到在不断开客户端连接的情况下更新业务代码。


## 范例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 设置此实例收到reload信号后是否重启
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// 运行worker
Worker::runAll();
```

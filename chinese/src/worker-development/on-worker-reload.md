# onWorkerReload
要求```（workerman >= 3.2.5）```
## 说明:
```php
callback Worker::$onWorkerReload
```

设置Worker收到reload信号后执行的回调。

可以利用onWorkerReload回调做很多事情，例如在不需要重启进程的情况下重新加载业务配置文件。

**注意**：

子进程收到reload信号默认的动作是退出重启，以便新进程重新加载业务代码完成代码更新。所以reload后子进程在执行完onWorkerReload回调后便立刻退出是正常现象。

如果在收到reload信号后只想让子进程执行onWorkerReload，不想退出，可以在初始化Worker实例时设置对应的Worker实例的reloadable属性为false。


## 回调函数的参数

``` $worker ```

即Worker对象



## 范例


```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 设置reloadable为false，即子进程收到reload信号不执行重启
$worker->reloadable = false;
// 执行reload后告诉所有客户端服务端执行了reload
$worker->onWorkerReload = function($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// 运行worker
Worker::runAll();
```

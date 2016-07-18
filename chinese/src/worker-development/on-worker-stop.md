# onWorkerStop
## 说明:
```php
callback Worker::$onWorkerStop
```

设置Workert停止时的回调函数，即当Worker收到stop信号后执行Worker::onWorkerStop指定的回调函数

## 回调函数的参数

``` $worker ```

即Worker对象

## 注意
如果业务代码发生致命错误(Fatal Error)或者进程被强行kill掉则不会触发onWorkerStop回调。

## 范例

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStop = function($worker)
{
    echo "Worker stopping...\n";
};
// 运行worker
Worker::runAll();
```

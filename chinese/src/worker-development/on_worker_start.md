# onWorkerStart
## 说明:
```php
callback Worker::$onWorkerStart
```

设置Worker启动时的回调函数，即当Worker启动后立即执行Worker::onWorkerStart成员指定的回调函数


## 回调函数的参数

``` $worker ```

即Worker对象



## 范例


```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// 运行worker
Worker::runAll();
```

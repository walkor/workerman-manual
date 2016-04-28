# count

## 说明:
```php
int Worker::$count
```

设置当前Worker实例启动多少个进程，不设置时默认为1。

如何设置进程数，请参考[**这里**](/faq/processes-count.html) 。


## 范例


```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 启动8个进程，同时监听8484端口，以websocket协议提供服务
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// 运行worker
Worker::runAll();
```

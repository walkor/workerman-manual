# onWorkerStart
## 说明:
```php
callback Worker::$onWorkerStart
```

设置Worker子进程启动时的回调函数，每个子进程启动时都会执行。

注意：onWorkerStart是在子进程启动时运行的，如果开启了多个子进程(```$worker->count > 1```)，每个子进程运行一次，则总共会运行```$worker->count```次。


## 回调函数的参数

 ``` $worker ```

即Worker对象



## 范例


```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "Worker {$worker->id} starting...\n";
};
// 运行worker
Worker::runAll();
```

> **提示**
> 业务可以根据worker->id来区分不同的进程从而执行不同的业务逻辑，例如只在0号进程执行某个业务，具体[参考这里](workerid.md)

> **提示**
> 除了使用匿名函数作为回调，还可以[参考这里](../faq/callback_methods.md)使用其它回调写法。

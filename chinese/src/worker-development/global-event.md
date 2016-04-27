# globalEvent

## 说明:
```php
static Event Worker::$globalEvent
```

此属性为全局静态属性，为全局的eventloop实例，可以向其注册文件描述符的读写事件或者信号事件。


## 范例

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once './Workerman/Autoloader.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'Pid is ' . posix_getpid() . "\n";
    // 当进程收到SIGALRM信号时，打印输出一些信息
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Get signal SIGALRM\n";
    });
};
// 运行worker
Worker::runAll();
```

## 测试
Workerman启动后会输出当前进程pid(一个数字)。命令行运行
```
kill -SIGALRM 进程pid
```
服务端会打印出
```
Get signal SIGALRM
```



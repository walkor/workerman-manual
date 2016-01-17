# stopAll
```php
void Worker::stopAll(void)
```

停止当前进程（子进程）的所有Worker实例并退出。

此方法用于安全退出当前子进程，作用相当于调用exit/die退出当前子进程。

与直接调用exit/die区别是，直接调用exit或者die无法触发onWorkerStop回调，并且会导致一条WORKER EXIT UNEXPECTED错误日志。

### 参数
无参数



### 返回值
无返回

## 范例 max_request

下面例子子进程每处理完1000个请求后执行stopAll退出，以便重新启动一个全新进程。类似php-fpm的max_request属性，主要用于解决php业务代码bug引起的内存泄露问题。

start.php

```php
<?php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

// 每个进程最多执行1000个请求
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function($connection, $data)
{
    // 已经处理请求数
    static $request_count = 0;

    $connection->send('hello http');
    // 如果请求数达到1000
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * 退出当前进程，主进程会立刻重新启动一个全新进程补充上来
         * 从而完成进程重启
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```

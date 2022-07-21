# stopAll
```php
void Worker::stopAll(void)
```

停止当前进程并退出。

> **注意**
> `Worker::stopAll()`用于停止当前进程，当前进程退出后主进程会立刻拉起一个新的进程。如果你想停止整个workerman服务，请调用`posix_kill(posix_getppid(), SIGINT)`

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
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 每个进程最多执行1000个请求
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
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

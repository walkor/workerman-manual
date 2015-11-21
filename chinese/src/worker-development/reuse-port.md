# reusePort
要求```（workerman >= 3.2.1 并且 PHP>=7.0）```

## 说明:

```php
bool Worker::$reusePort
```

设置当前worker是否开启监听端口复用(socket的SO_REUSEPORT选项)，默认为false，不开启。

开启监听端口复用后允许多个无亲缘关系的进程监听相同的端口，并且由系统内核做负载均衡，决定将socket连接交给哪个进程处理，避免了惊群效应，可以提升多进程短连接应用的性能。

**注意：**此特性需要PHP版本>=7.0


## 范例 1

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function($connection, $data)
{
    $connection->send('ok');
};
// 运行worker
Worker::runAll();
```

## 范例2：workerman多端口(多协议)监听
```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// 每个进程启动后在当前进程新增一个监听
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * 多个进程监听同一个端口（监听套接字不是继承自父进程）
     * 需要开启端口复用，不然会报Address already in use错误
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // 执行监听
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message($connection, $data)
{
    $connection->send("hello\n");
}

// 运行worker
Worker::runAll();
```

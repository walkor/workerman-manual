# 构造函数 __construct

## 说明:
```php
Worker::__construct([$listen , $context])
```

初始化一个Worker容器实例，可以设置容器的一些属性和回调接口，完成特定功能。


## 参数
``` $listen ```


如果有设置监听$listen参数，则会执行socket监听。

``` $listen ```的格式为 <协议>://<监听地址>

协议：可以是tcp、udp传输层协议，或者unix采用unix domain，或者直接指定应用层协议例如http、webscoket



## 范例


```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function($connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// 运行worker
Worker::runAll();
```

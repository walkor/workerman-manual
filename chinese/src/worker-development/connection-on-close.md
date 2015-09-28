# onClose
## 说明:
```php
callback Connection::$onClose
```

此回调与```Worker::$onClose```回调作用相同，区别是只针对当前连接有效,也就是可以针对某个连接的设置onClose回调。

## 范例

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 当有链接事件时触发
$worker->onConnect = function($connection)
{
    // 设置连接的onClose回调
    $connection->onClose = function($connection)
    {
        echo "connection closed\n";
    };
};
// 运行worker
Worker::runAll();
```

上面代码与下面的效果相同

```php
use Workerman\Worker;
$worker = new Worker('websocket://0.0.0.0:8484');
// 设置所有连接的onclose回调
$worker->onClose = function($connection)
{
    echo "connection closed\n";
};
// 运行worker
Worker::runAll();
```

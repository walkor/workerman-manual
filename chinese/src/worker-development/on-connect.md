# onConnect
## 说明:
```php
callback Worker::$onConnect
```

当连接建立时触发的回调函数。每个连接只会触发一次```onConnect```回调。

## 回调函数的参数

``` $connection ```

连接对象，连接对象的说明见下一节


## 范例

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    echo "new connection from ip " . $connection->getRemoteIp() . "\n";
};
// 运行worker
Worker::runAll();
```

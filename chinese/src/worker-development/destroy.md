# destroy
## 说明:
```php
void Connection::destroy()
```

立刻关闭连接。

与close不同之处是，调用destroy后即使该连接的发送缓冲区还有数据未发送到对端，连接也会立刻被关闭，并立刻触发该连接的```onClose```回调。

## 参数

无参数


## 范例

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function($connection, $data)
{
    // if something wrong
    $connection->destroy();
};
// 运行worker
Worker::runAll();
```

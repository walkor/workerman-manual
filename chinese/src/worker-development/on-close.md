# onClose
## 说明:
```php
callback Worker::$onClose
```

当客户端的连接断开时触发，不管连接是如何断开的，只要断开就会触发

## 回调函数的参数

``` $connection ```

连接对象，连接对象的说明见下一节


## 范例

```php
use Workerman\Worker;
$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function($connection)
{
    echo "connection closed\n";
};
// 运行worker
Worker::runAll();
```

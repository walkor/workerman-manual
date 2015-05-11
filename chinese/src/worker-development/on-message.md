# onMessage
## 说明:
```php
callback Worker::$onMessage
```

当有客户端的连接上有数据发来时触发

## 回调函数的参数

``` $connection ```

连接对象，连接对象的说明见下一节

``` $data ```

客户端连接上发来的数据，如果Worker指定了协议，则$data是对应协议decode（解码）了的数据


## 范例

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function($connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// 运行worker
Worker::runAll();
```

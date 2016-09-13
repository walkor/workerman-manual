# onClose
## 说明:
```php
callback Worker::$onClose
```

当连接断开时触发的回调函数。不管连接是如何断开的，只要断开就会触发```onClose```。每个连接只会触发一次```onClose```。

注意：如果对端是由于断网或者断电等极端情况断开的连接，这时由于无法及时发送tcp的fin包给workerman，workerman就无法得知连接已经断开，也就无法及时触发```onClose```。这种情况需要通过应用层心跳来解决。workerman中连接的心跳实现参见[这里](/faq/heartbeat.html)。如果使用的是GatewayWorker框架，则直接使用GatewayWorker框架的心跳机制即可，参见[这里](http://workerman.net/gatewaydoc/gateway-worker-development/heartbeat.html)。

## 回调函数的参数

``` $connection ```

连接对象，连接对象的说明见下一节


## 范例

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function($connection)
{
    echo "connection closed\n";
};
// 运行worker
Worker::runAll();
```

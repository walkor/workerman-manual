# send
## 说明:
```php
mixed Connection::send(mixed $data [,$raw = false])
```

向客户端发送数据

## 参数

``` $data ```

要发送的数据，如果在初始化Worker类时指定了协议，则会自动调用协议的encode方法,完成协议打包工作后发送给客户端

``` $raw ```
是否发送原始数据，即不调用协议的encode方法，默认是false，即自动调用协议的encode方法

## 返回值

true 表示发送成功

null 表示放入待发送队列，等待异步发送

false 表示发送失败，失败原因可能是客户端连接已经关闭，或者该连接的应用层发送缓冲区已满


## 范例

```php
use Workerman\Worker;
$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function($connection, $data)
{
    // 会自动调用\Workerman\Protocols\Websocket::encode打包成websocket协议数据后发送
    $connection->send("hello\n");
};
```

# onError
## 说明:
```php
callback Worker::$onError
```

当客户端的连接上发生错误时触发。

目前错误类型有

1、调用Connection::send由于客户端连接断开导致的失败 ```
(code:WORKERMAN_SEND_FAIL msg:client closed)```


2、在触发onBufferFull后，仍然调用Connection::send，并且发送缓冲区仍然是满的状态```，导致发送失败
(code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package)```


3、使用AsyncTcpConnection异步连接失败时 ```
(code:WORKERMAN_CONNECT_FAIL msg:stream_socket_client返回的错误消息)```


## 回调函数的参数

``` $connection ```

连接对象，连接对象的说明见下一节

``` $code ```

错误码

``` $msg ```

错误消息


## 范例

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function($connection, $code, $msg)
{
    echo "error $code $msg\n";
};
// 运行worker
Worker::runAll();
```

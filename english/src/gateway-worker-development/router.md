# router

## 说明:
```php
callback Gateway::$router
```

设置Gateway到BusinessWorker路由规则。不设置默认是Gateway随机选择一个BusinessWorker进程把数据转发给它处理。

期待该回调函数从所有到BusinessWorker进程的连接对象中选择一个并返回。


## 回调函数的参数

``` $worker_connnections ```

是一个数组，里面包含了所有到BusinessWorker进程的连接对象。数组的key为BusinessWorker进程的通讯地址，格式为ip:port。回调函数最终将返回该数组中一个连接对象。


``` $client_connection ```

客户端连接对象,可以通过此对象获得客户端ip端口等信息

``` $cmd ```

当前什么类型的消息，是个数字，分别可能为

1: CMD_ON_CONNECTION，即连接事件

2: CMD_ON_MESSAGE，即消息事件

3: CMD_ON_CLOSE，即客户端关闭事件


``` $buffer ```

客户端发来的数据。注意只有当 ``` $cmd ``` 为``` 2 ```时 ```$buffer ```才有值

## 返回值
返回 ```$worker_connnections``` 中的一个连接对象



## 范例

```php
use \GatewayWorker\Gateway;
$gateway = new Gateway("Websocket://0.0.0.0:8585");
// 随机路由
$gateway->router = function($worker_connections, $client_connection, $cmd, $buffer)
{
    return $worker_connections[array_rand($worker_connections)];
};
```

# Event::onConnect

## 说明:
```php
void Event::onConnect(int $client_id);
```

当客户端连接上gateway进程时触发。

绝大多数应用不用实现这个函数。

## 参数

``` $client_id ```
全局唯一的客户端socket_id

## 返回值
无返回值，任何返回值都会被视为无效的


## 范例
```php
use \GatewayWorker\Lib\Gateway;

class Event
{
    ...

    public onConnect($client)
    {
       Gateway::sendToCurrentClient('hello');
    }

    ...

}
```

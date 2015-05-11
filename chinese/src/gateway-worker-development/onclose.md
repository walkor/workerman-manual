# Event::onClose

## 说明:
```php
void Event::onClose(int $client_id);
```

客户端主动断开时触发。一般在这里做一些数据清理工作

## 参数
``` $client_id ```

全局唯一的client_id

## 返回值
无返回值，任何返回值都会被视为无效的

## 范例

```php
use \GatewayWorker\Lib\Gateway;

class Event
{
    ...

    /**
     * 当用户断开连接时触发的方法
     * @param integer $client_id 断开连接的客户端client_id
     * @return void
     */
    public static function onClose($client_id)
    {
       // 广播 xxx 退出了
       GateWay::sendToAll("client[$client_id] logout\n"));
    }

    ...
}
```

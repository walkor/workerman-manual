# \GatewayWorker\Lib\Gateway::sendToClient

## 说明:
```php
void Gateway::sendToClient(int $client_id, mixed $send_data);
```

向客户端client_id发送```$send_data```数据。如果client_id对应的客户端不存在或者不在线则自动丢弃发送数据

## 参数

* ```$client_id```

客户端的client_id，当客户端连接Gateway的那一刻框架便为其分配了一个全局唯一的client_id用来全局标识一个客户端连接。对某个客户端的操作都需要知道客户端的client_id

* ```$send_data```

要发送的数据，此数据会被Gateway所使用协议的encode方法打包后再发送给客户端

## 范例
```php
use \GatewayWorker\Lib\Gateway;
class Event
{
    ...

    public static function onMessage($client_id, $message)
    {
        // $message = '{"type":"say_to_one","to_client_id":100,"content":"hello"}'
        $req_data = json_decode($message, true);
        // 如果是向某个客户端发送消息
        if($req_data['type'] == 'say_to_one')
        {
            // 转发消息给对应的客户端
            Gateway::sendToClient($req_data['to_client_id'], $req_data['content']);
        }
    }

    ...
}

```

# \GatewayWorker\Lib\Gateway::sendToAll

## 说明:
```php
void Gateway::sendToAll(mixed $send_data [, array $client_id_array=array()]);
```

向所有客户端或者client_id_array指定的客户端发送```$send_data```数据。如果指定的$client_id_array中的client_id不存在则自动丢弃


## 参数

* ```$send_data```

要发送的数据，此数据会被Gateway所使用协议的encode方法打包后发送给客户端


* ```$client_id_array```

指定向哪些client_id发送，如果不传递该参数，则是向所有在线客户端发送 ```$send_data``` 数据

## 范例
```php
use \GatewayWorker\Lib\Gateway;

class Event
{
    ...

    public static function onMessage($client_id, $message)
    {
        // $message = '{"type":"say_to_all","content":"hello"}'
        $req_data = json_decode($message, true);
        // 如果是向所有客户端发送消息
        if($req_data['type'] == 'say_to_all')
        {
            Gateway::sendToAll($req_data['content']);
        }
    }
    ...
}

```

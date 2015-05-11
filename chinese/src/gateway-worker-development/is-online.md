# \GatewayWorker\Lib\Gateway::isOnline

## 说明:
```php
int Gateway::isOnline(int $client_id);
```

判断$client_id是否还在线


## 参数

* ```$client_id```

全局唯一的客户端client_id

## 返回值
在线返回1，不在线返回0


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
        if($req_data['type'] == 'say_to_one'))
        {
            // 如果不在线就先存起来
            if(!Gateway::isOnline($req_data['to_client_id'])
            {
                your_store_fun($message);
            }
            else
            {
                // 在线就转发消息给对应的客户端
                Gateway::sendToClient($req_data['to_client_id'], $req_data['content']);
            }
        }
    }

    ...
}

```


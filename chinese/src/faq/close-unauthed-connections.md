# 关闭未认证连接
**问题：**

如何关闭规定时间内未发送过数据的客户端,<br>
比如30秒内没收到一条数据就自动关闭这个客户端连接,<br>
目的是为了让未认证的连接必须在规定时间内认证

**答案：**

```php
use Workerman\Lib\Timer;
$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function($connection)
{
    // 临时给$connection对象添加一个auth_timer_id属性存储定时器id
    // 定时30秒关闭连接，需要客户端30秒内发送验证删除定时器
    $connection->auth_timer_id = Timer::add(30, function()use($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function($connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
    case 'login':
        ...略
        // 验证成功，删除定时器，防止连接被关闭
        Timer::del($connection->auth_timer_id);
        break;
         ... 略
    }
    ... 略
}
```

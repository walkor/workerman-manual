# WorkerMan中如何向某个特定客户端发送数据
使用worker来做服务器，没有用GatewayWorker，如何实现向指定用户推送消息？

```php
<?php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';
// 初始化一个worker容器，监听1234端口
$worker = new Worker('websocket://workerman.net:1234');
// 进程数设置为1
$worker->count = 1;
// 新增加一个属性，用来保存uid到connection的映射
$worker->uidConnections = array();
// 当有客户端发来消息时执行的回调函数
$worker->onMessage = function($connection, $data)
{
    global $worker;
    // 判断当前客户端是否已经验证,即是否设置了uid
    if(!isset($connection->uid))
    {
       // 没验证的话把第一个包当做uid（这里为了方便演示，没做真正的验证）
       $connection->uid = $data;
       /* 保存uid到connection的映射，这样可以方便的通过uid查找connection，
        * 实现针对特定uid推送数据
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return $connection->send('login success, your uid is ' . $connection->uid);
    }
    // 其它逻辑，针对某个uid发送 或者 全局广播
    // 假设消息格式为 uid:message 时是对 uid 发送 message
    // uid 为 all 时是全局广播
    list($recv_uid, $message) = explode(':', $data);
    // 全局广播
    if($recv_uid == 'all')
    {
        broadcast($message);
    }
    // 给特定uid发送
    else
    {
        sendMessageByUid($recv_uid, $message);
    }
};

// 当有客户端连接断开时
$worker->onClose = function($connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // 连接断开时删除映射
        unset($worker->uidConnections[$connection->uid]);
    }
};

// 向所有验证的用户推送数据
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// 针对uid推送数据
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
    }
}

// 运行所有的worker（其实当前只定义了一个）
Worker::runAll();
```
**说明：**

以上例子可以针对uid推送，虽然是单进程，但是支持个10W在线是没问题的。


注意这个例子只能单进程，要支持多进程或者服务器集群的话需要Channel组件完成进程间通讯，开发也非常简单，可以参考[Channel组件集群推送例子](http://doc3.workerman.net/component/channel-examples.html)一节。

**如果希望在其它系统中推送消息给客户端，可以参考[在其它项目中推送](http://doc3.workerman.net/faq/push-in-other-project.html)一节**

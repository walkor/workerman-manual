# 如何实现异步任务

**问：**

如何异步处理繁重的业务，避免主业务被长时间阻塞。例如我要给1000用户发送邮件，这个过程很慢，可能要阻塞数秒，这个过程中因为主流程被阻塞，会影响后续的请求，如何将这样的繁重任务交给其它进程异步处理。

**答:**

可以在本机或者其它服务器甚至服务器集群预先建立一些任务进程处理繁重的业务，任务进程数可以开多一些，例如cpu的10倍，然后调用方利用AsyncTcpConnection将数据异步发送给这些任务进程异步处理，异步得到处理结果。

任务进程服务端
```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';
// task worker，使用Text协议
$task_worker = new Worker('Text://0.0.0.0:12345');
// task进程数可以根据需要多开一些
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function($connection, $task_data)
{
     // 假设发来的是json数据
     $task_data = json_decode($task_data, true);
     // 根据task_data处理相应的任务逻辑.... 得到结果，这里省略....
     $task_result = ......
     // 发送结果
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

在workerman中调用

```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
require_once './Workerman/Autoloader.php';

// websocket服务
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function($ws_connection, $message)
{
    // 与远程task服务建立异步链接，ip为远程task服务的ip，如果是本机就是127.0.0.1，如果是集群就是lvs的ip
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // 任务及参数数据
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // 发送数据
    $task_connection->send(json_encode($task_data));
    // 异步获得结果
    $task_connection->onMessage = function($task_connection, $task_result)use($ws_connection)
    {
         // 结果
         var_dump($task_result);
         // 获得结果后记得关闭异步链接
         $task_connection->close();
         // 通知对应的websocket客户端任务完成
         $ws_connection->send('task complete');
    };
    // 执行异步链接
    $task_connection->connect();
}

Worker::runAll();
```

这样，繁重的任务交给本机或者其它服务器的进程去做，任务完成后会异步收到结果，业务进程就不会阻塞了。




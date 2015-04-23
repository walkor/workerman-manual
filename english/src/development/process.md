# 基本流程
（以一个简单的Websocket聊天室服务端为例）

#### 1、在applications下建立项目目录，例如建立目录
Applications/SimpleChat/

#### 2、选定开发模型，基于Worker开发还是基于Worker/Gateway开发
假设用户量不大，这里基于Worker开发

#### 3、选定协议
这里我们选定Text文本协议(WorkerMan中自定义的一个协议，格式为文本+换行)

（目前WorkerMan支持HTTP、Websocket、Text文本协议，如果需要使用其它协议，请参照协议一章开发自己的协议）

#### 4、写启动脚本
Applications/SimpleChat/start.php
```php
<?php
use Workerman\Worker;

$global_uid = 0;
$connections_array = array();

// 当客户端连上来时分配uid，并保存连接，并通知所有客户端
function handler_connection($connection)
{
    global $connections_array, $global_uid;
    // 为这个链接分配一个uid
    $connection->uid = ++$global_uid;
    // 保存连接
    $connections_array[$connection->uid] = $connection;
}

// 当客户端发送消息过来时，转发给所有人
function handle_message($connection, $data)
{
    global $connections_array;
    foreach($connections_array as $conn)
    {
        $conn->send("user[{$connection->uid}] said: $data");
    }
}

// 当客户端断开时，从连接数组中删除
function handle_close($connection)
{
    global $connections_array;
    unset($connections_array[$connection->uid]);
}


// 创建一个文本协议的Worker监听2347接口
$text_worker = new Worker("Text://0.0.0.0:2347");

// 只启动1个进程，这样方便客户端之间传输数据
$text_worker->count = 1;

$text_worker->onConnect = 'handler_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

```

#### 5、测试
Text协议可以用telnet命令测试
```shell
telnet 127.0.0.1 2347
```

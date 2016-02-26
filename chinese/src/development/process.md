# 基本流程
（以一个简单的Websocket聊天室服务端为例）

#### 1、任意位置建立项目目录
如 SimpleChat/

#### 2、引入Workerman/Autoloader.php
如
```php
require_once '/your/path/Workerman/Autoloader.php';
```

#### 3、选定协议
这里我们选定Text文本协议(WorkerMan中自定义的一个协议，格式为文本+换行)

（目前WorkerMan支持HTTP、Websocket、Text文本协议，如果需要使用其它协议，请参照协议一章开发自己的协议）

#### 4、根据需要写入口启动脚本
例如下面这个是一个简单的聊天室的入口文件。

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
require_once '/your/path/Workerman/Autoloader.php';

$global_uid = 0;

// 当客户端连上来时分配uid，并保存连接，并通知所有客户端
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // 为这个链接分配一个uid
    $connection->uid = ++$global_uid;
}

// 当客户端发送消息过来时，转发给所有人
function handle_message($connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] said: $data");
    }
}

// 当客户端断开时，广播给所有客户端
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] logout");
    }
}

// 创建一个文本协议的Worker监听2347接口
$text_worker = new Worker("text://0.0.0.0:2347");

// 只启动1个进程，这样方便客户端之间传输数据
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();

```

#### 5、测试
Text协议可以用telnet命令测试
```shell
telnet 127.0.0.1 2347
```

# 构造函数 __construct

## 说明:
```php
Worker::__construct([string $listen , array $context])
```

初始化一个Worker容器实例，可以设置容器的一些属性和回调接口，完成特定功能。


## 参数
#### **``` $listen ```**



如果有设置监听```$listen```参数，则会执行socket监听。

``` $listen ```的格式为 <协议>://<监听地址>

**<协议> 可以为以下格式：**

tcp: 例如 ```tcp://0.0.0.0:8686```

udp: 例如 ```udp://0.0.0.0:8686```

unix: 例如 ```unix:///tmp/my_file ``` ```(需要Workerman>=3.2.7)```

http: 例如 ```http://0.0.0.0:80```

websocket: 例如 ```websocket://0.0.0.0:8686```

text: 例如 ```text://0.0.0.0:8686``` ```(text是Workerman内置的文本协议，兼容telnet，详情参见附录Text协议部分)```

以及其他自定义协议，参见本手册定制通讯协议部分

**<监听地址> 可以为以下格式：**

如果是unix套接字，地址为本地一个磁盘路径

非unix套接字，地址格式为 <本机ip>:<端口号>

<本机ip>可以为```0.0.0.0```表示监听本机所有网卡，包括内网ip和外网ip及本地回环127.0.0.1

<本机ip>如果以为```127.0.0.1```表示监听本地回环，只能本机访问，外部无法访问

<本机ip>如果为内网ip，类似```192.168.xx.xx```，表示只监听内网ip，则外网用户无法访问

<本机ip>设置的值不属于本机ip则无法执行监听，并且提示```Cannot assign requested address```错误

**注意：**<端口号>不能大于65535。<端口号>如果小于1024则需要root权限才能监听。监听的端口必须是本机未被占用的端口，否则无法监听，并且提示```Address already in use```错误


#### **``` $context ```**


一个数组。用于传递socket的上下文选项，参见[套接字上下文选项](http://php.net/manual/zh/context.socket.php)


## 范例

Worker作为http容器监听处理http请求
```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function($connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// 运行worker
Worker::runAll();
```

Worker作为websocket容器监听处理websocket请求
```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8686');

$worker->onMessage = function($connection, $data)
{
    $connection->send("hello");
};

// 运行worker
Worker::runAll();
```

Worker作为tcp容器监听处理tcp请求
```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('tcp://0.0.0.0:8686');

$worker->onMessage = function($connection, $data)
{
    $connection->send("hello");
};

// 运行worker
Worker::runAll();
```

Worker作为udp容器监听处理udp请求
```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('udp://0.0.0.0:8686');

$worker->onMessage = function($connection, $data)
{
    $connection->send("hello");
};

// 运行worker
Worker::runAll();
```

Worker监听unix domain套接字```(要求Workerman版本>=3.2.7)```
```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('unix:///tmp/my.sock');

$worker->onMessage = function($connection, $data)
{
    $connection->send("hello");
};

// 运行worker
Worker::runAll();
```


不执行任何监听的Worker容器，用来处理一些定时任务
```php
use \Workerman\Worker;
use \Workerman\Lib\Timer;
require_once './Workerman/Autoloader.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // 每2.5秒执行一次
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// 运行worker
Worker::runAll();
```

**Worker监听自定义协议的端口**

最终的目录结构
```
├── Protocols              // 这是要创建的Protocols目录
│   └── MyTextProtocol.php // 这是要创建的自定义协议文件
├── test.php  // 这次要创建的test脚本
└── Workerman // Workerman源码目录，里面代码不要动
```

1、创建Protocols目录，并创建一个协议文件
Protocols/MyTextProtocol.php（参照上面目录结构）

```php
// 用户自定义协议命名空间统一为Protocols
namespace Protocols;
//简单文本协议，协议格式为 文本+换行
class MyTextProtocol
{
    // 分包功能，返回当前包的长度
    public static function input($recv_buffer)
    {
        // 查找换行符
        $pos = strpos($recv_buffer, "\n");
        // 没找到换行符，表示不是一个完整的包，返回0继续等待数据
        if($pos === false)
        {
            return 0;
        }
        // 查找到换行符，返回当前包的长度，包括换行符
        return $pos+1;
    }

    // 收到一个完整的包后通过decode自动解码，这里只是把换行符trim掉
    public static function decode($recv_buffer)
    {
        return trim($recv_buffer);
    }

    // 给客户端send数据前会自动通过encode编码，然后再发送给客户端，这里加了换行
    public static function encode($data)
    {
        return $data."\n";
    }
}
```
2、使用MyTextProtocol协议监听处理请求

参照上面最终目录结构创建test.php文件

```php
require_once './Workerman/Autoloader.php';
use Workerman\Worker;

// #### MyTextProtocol worker ####
$text_worker = new Worker("MyTextProtocol://0.0.0.0:5678");

/*
 * 收到一个完整的数据（结尾是换行）后，自动执行MyTextProtocol::decode('收到的数据')
 * 结果通过$data传递给onMessage回调
 */
$text_worker->onMessage =  function($connection, $data)
{
    var_dump($data);
    /*
     * 给客户端发送数据，会自动调用MyTextProtocol::encode('hello world')进行协议编码，
     * 然后再发送到客户端
     */
    $connection->send("hello world");
};

// run all workers
Worker::runAll();
```

3、测试

打开终端，进入到test.php所在目录，执行```php test.php start```
```
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

打开终端，利用telnet测试（建议用linux系统的telnet）

假设是本机测试，
终端执行 telnet 127.0.0.1 5678
然后输入 hi回车
会接收到数据hello world\n
```
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world

```

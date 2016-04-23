# __construct
```php
void \Workerman\Connection\AsyncTcpConnection::__construct(string $remote_address)
```
创建一个异步连接对象

### 参数
``` remote_address ```

连接的地址，例如<br>
``` tcp://www.baidu.com:80 ```<br>
``` ws://echo.websocket.org:80 ```<br>
``` frame://192.168.1.1:8080 ```<br>
``` text://192.168.1.1:8080 ```<br>

注意：

目前AsyncTcpConnection支持的协议有[tcp](http://baike.baidu.com/subview/32754/8048820.htm)、[ws](/appendices/about-ws.html)、[frame](/appendices/about-frame.html)、[text](/appendices/about-text.html)。

目前不支持[http](http://baike.baidu.com/view/9472.htm)协议的AsyncTcpConnection。

可以用```new AsyncTcpConnection('ws://...')```像浏览器一样在workerman里发起websocket连接远程websocket服务器，见[示例](/appendices/about-ws.html)。但是不能以 ```new AsyncTcpConnection('websocket://...')```的形式在workerman里发起websocket连接。


### 返回值
无返回值

### 示例
```php
use \Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
require_once './Workerman/Autoloader.php';

$task = new Worker();
// 进程启动时异步建立一个到www.baidu.com连接对象，并发送数据获取数据
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // 当连接建立成功时，发送http请求数据
    $connection_to_baidu->onConnect = function($connection_to_baidu)
    {
        echo "connect success\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function($connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function($connection_to_baidu)
    {
        echo "connection closed\n";
    };
    $connection_to_baidu->onError = function($connection_to_baidu, $code, $msg)
    {
        echo "Error code:$code msg:$msg\n";
    };
    $connection_to_baidu->connect();
};

// 运行worker
Worker::runAll();
```


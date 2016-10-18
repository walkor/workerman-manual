# transport属性
```要求（workerman >= 3.3.4）```

设置传输属性，可选值为 [tcp](http://baike.baidu.com/subview/32754/8048820.htm) 和 [ssl](http://baike.baidu.com/view/525499.htm)，默认是tcp。

transport为 [ssl](http://baike.baidu.com/view/525499.htm) 时，要求PHP必须安装[openssl扩展](http://php.net/manual/zh/book.openssl.php)。


当把Workerman作为客户端向服务端发起ssl加密连接(https连接、wss连接等)时请设置此选项为```ssl```，例如下面的例子。



### 示例 (https连接)
```php
use \Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
require_once './Workerman/Autoloader.php';

$task = new Worker();
// 进程启动时异步建立一个到www.baidu.com连接对象，并发送数据获取数据
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // 设置为ssl加密连接
    $connection_to_baidu->transport = 'ssl';

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

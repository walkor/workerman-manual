# __construct 方法
```php
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
创建一个异步连接对象。

AsyncTcpConnection可以让Workerman作为客户端向远程服务端发起异步连接，并通过send接口和onMessage回调异步发送和处理连接上的数据。

## 参数
参数:``` remote_address ```

连接的地址，例如
 ``` tcp://www.baidu.com:80 ```
 ``` ssl://www.baidu.com:443 ```
 ``` ws://echo.websocket.org:80 ```
 ``` frame://192.168.1.1:8080 ```
 ``` text://192.168.1.1:8080 ```


参数:``` $context_option ```

 ```此参数要求（workerman >= 3.3.5）```


用来设置socket上下文，例如利用```bindto```设置以哪个(网卡)ip和端口访问外部网络，设置ssl证书等。

参考 [stream_context_create](https://php.net/manual/en/function.stream-context-create.php)、 [套接字上下文选项](https://php.net/manual/zh/context.socket.php)、[SSL 上下文选项](https://php.net/manual/zh/context.ssl.php)

## 注意

目前AsyncTcpConnection支持的协议有[tcp](https://baike.baidu.com/subview/32754/8048820.htm)、[ssl](https://baike.baidu.com/view/525499.htm)、[ws](appendices/about-ws.md)、[frame](appendices/about-frame.md)、[text](appendices/about-text.md)。

同时支持自定义协议，参见[如何自定义协议](../protocols/how-protocols.md)

其中[ssl](https://baike.baidu.com/view/525499.htm)要求Workerman>=3.3.4，并安装[openssl扩展](https://php.net/manual/zh/book.openssl.php)。

目前不支持[http](https://baike.baidu.com/view/9472.htm)协议的AsyncTcpConnection。

可以用```new AsyncTcpConnection('ws://...')```像浏览器一样在workerman里发起websocket连接远程websocket服务器，见[示例](../appendices/about-ws.md)。但是不能以 ```new AsyncTcpConnection('websocket://...')```的形式在workerman里发起websocket连接。


## 示例

### 示例 1、异步访问外部http服务
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// 进程启动时异步建立一个到www.baidu.com连接对象，并发送数据获取数据
$task->onWorkerStart = function($task)
{
    // 不支持直接指定http，但是可以用tcp模拟http协议发送数据
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // 当连接建立成功时，发送http请求数据
    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "connect success\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "connection closed\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Error code:$code msg:$msg\n";
    };
    $connection_to_baidu->connect();
};

// 运行worker
Worker::runAll();
```

### 示例 2、异步访问外部websocket服务，并设置以哪个本地ip及端口访问
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker) {
    // 如果你想只在一个进程里发起连接，可以通过判断$worker->id来做到，例如下面是只在0号进程发起连接
    if ($worker->id != 0) {
        return;
    }
    // 设置访问对方主机的本地ip及端口(每个socket连接都会占用一个本地端口)
    $context_option = array(
        'socket' => array(
            // ip必须是本机网卡ip，并且能访问对方主机，否则无效
            'bindto' => '114.215.84.87:2333',
        ),
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80', $context_option);

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```


### 示例 3、异步访问外部wss端口，并设置本地ssl证书
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // 设置访问对方主机的本地ip及端口以及ssl证书
    $context_option = array(
        'socket' => array(
            // ip必须是本机网卡ip，并且能访问对方主机，否则无效
            'bindto' => '114.215.84.87:2333',
        ),
        // ssl选项，参考https://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // 本地证书路径。 必须是 PEM 格式，并且包含本地的证书及私钥。
            'local_cert'        => '/your/path/to/pemfile',
            // local_cert 文件的密码。
            'passphrase'        => 'your_pem_passphrase',
            // 是否允许自签名证书。
            'allow_self_signed' => true,
            // 是否需要验证 SSL 证书。
            'verify_peer'       => false
        )
    );

    // 发起异步连接
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // 设置以ssl加密方式访问
    $con->transport = 'ssl';

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```




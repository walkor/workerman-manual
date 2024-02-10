# __construct 方法
```php
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
建立一個異步連接物件。

AsyncTcpConnection 可讓 Workerman 作為客戶端向遠程服務端發起異步連接，並通過 send 介面和 onMessage 回調異步發送和處理連接上的數據。

## 參數
參數：``` remote_address ```

連接的地址，例如
 ``` tcp://www.baidu.com:80 ```
 ``` ssl://www.baidu.com:443 ```
 ``` ws://echo.websocket.org:80 ```
 ``` frame://192.168.1.1:8080 ```
 ``` text://192.168.1.1:8080 ```


參數：``` $context_option ```

 ```此參數要求（workerman >= 3.3.5）```


用來設置 socket 上下文，例如利用```bindto```設置以哪個(網卡)ip和端口訪問外部網絡，設置ssl證書等。

參考 [stream_context_create](https://php.net/manual/en/function.stream-context-create.php)、 [套接字上下文選項](https://php.net/manual/zh/context.socket.php)、[SSL 上下文選項](https://php.net/manual/zh/context.ssl.php)

## 注意

目前 AsyncTcpConnection 支持的協議有 [tcp](https://baike.baidu.com/subview/32754/8048820.htm)、[ssl](https://baike.baidu.com/view/525499.htm)、[ws](appendices/about-ws.md)、[frame](appendices/about-frame.md)、[text](appendices/about-text.md)。

同時支持自定義協議，請參見 [如何自定義協議](../protocols/how-protocols.md)

其中 [ssl](https://baike.baidu.com/view/525499.htm) 要求 Workerman>=3.3.4，並安裝 [openssl擴展](https://php.net/manual/zh/book.openssl.php)。

目前不支持 [http](https://baike.baidu.com/view/9472.htm) 協議的 AsyncTcpConnection。

可以用```new AsyncTcpConnection('ws://...')```像瀏覽器一樣在 Workerman 里發起 websocket 連接遠程 websocket 服務器，請參見 [示例](../appendices/about-ws.md)。但是不能以 ```new AsyncTcpConnection('websocket://...')```的形式在 Workerman 里發起 websocket 連接。


## 示例

### 示例 1、異步訪問外部http服務
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// 進程啟動時異步建立一個到www.baidu.com連接物件，並發送數據獲取數據
$task->onWorkerStart = function($task)
{
    // 不支持直接指定 http，但是可以用 tcp 模擬 http 協議發送數據
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // 當連接建立成功時，發送http請求數據
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

// 運行 worker
Worker::runAll();
```

### 示例 2、異步訪問外部websocket服務，並設置以哪個本地ip及端口訪問
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // 設置訪問對方主機的本地ip及端口(每個 socket 連接都會佔用一個本地端口)
    $context_option = array(
        'socket' => array(
            // ip必須是本機網卡 ip，並且能訪問對方主機，否則無效
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


### 示例 3、異步訪問外部wss端口，並設置本地ssl證書
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // 設置訪問對方主機的本地ip及端口以及ssl證書
    $context_option = array(
        'socket' => array(
            // ip必須是本機網卡 ip，並且能訪問對方主機，否則無效
            'bindto' => '114.215.84.87:2333',
        ),
        // ssl選項，參考https://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // 本地證書路徑。 必須是 PEM 格式，並且包含本地的證書及私鑰。
            'local_cert'        => '/your/path/to/pemfile',
            // local_cert 文件的密碼。
            'passphrase'        => 'your_pem_passphrase',
            // 是否允許自簽名證書。
            'allow_self_signed' => true,
            // 是否需要驗證 SSL 證書。
            'verify_peer'       => false
        )
    );

    // 發起異步連接
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // 設置以ssl加密方式訪問
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

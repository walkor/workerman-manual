# transport屬性
```要求（workerman >= 3.3.4）```

設置傳輸屬性，可選值為 [tcp](https://baike.baidu.com/subview/32754/8048820.htm) 和 [ssl](https://baike.baidu.com/view/525499.htm)，默認是tcp。

當transport為 [ssl](https://baike.baidu.com/view/525499.htm) 時，要求PHP必須安裝[openssl擴展](https://php.net/manual/zh/book.openssl.php)。

當將Workerman作為客戶端向服務端發起ssl加密連接(https連接、wss連接等)時請設置此選項為```ssl```，例如下面的例子。

### 示例 (https連接)
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// 進程啟動時異步建立一個到www.baidu.com連接對象，並發送數據獲取數據
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // 設置為ssl加密連接
    $connection_to_baidu->transport = 'ssl';

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

// 運行worker
Worker::runAll();
```

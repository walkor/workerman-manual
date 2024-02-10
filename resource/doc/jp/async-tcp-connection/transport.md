# transportプロパティ
```要求（workerman >= 3.3.4）```

transportプロパティは、[tcp](https://baike.baidu.com/subview/32754/8048820.htm) と [ssl](https://baike.baidu.com/view/525499.htm) のオプション値を設定することができます。デフォルトはtcpです。

transportを [ssl](https://baike.baidu.com/view/525499.htm) に設定する場合、PHPには[openssl拡張](https://php.net/manual/zh/book.openssl.php)が必要です。

Workermanをクライアントとして使用し、ssl暗号化された接続（https接続、wss接続など）をサーバーに要求する場合は、このオプションを`ssl`に設定してください。以下はその例です。

### 例（https接続）
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// ワーカーが起動すると、www.baidu.comへの接続を非同期で確立し、データを送信してデータを取得します
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // ssl暗号化接続に設定
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

// ワーカーを実行
Worker::runAll();
```

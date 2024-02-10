# 基本偵錯

WorkerMan有兩種運行模式，偵錯模式以及守護進程運行模式。

執行 ```php start.php start``` 進入偵錯模式，這時程式碼中的 ```echo、var_dump、var_export``` 等函數打印將會在終端顯示。請注意，以 ```php start.php start``` 運行的WorkerMan在終端關閉時所有進程將會退出。

而執行 ```php start.php start -d``` 則是進入守護進程模式，也就是正式上線的運行模式，關閉終端不受影響。

如果想要在守護方式運行時也能看到 ```echo、var_dump、var_export``` 等函數打印，可以設定Worker::$stdoutFile屬性，例如

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 將螢幕打印輸出到Worker::$stdoutFile指定的檔案中
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```

這樣所有的 ```echo、var_dump、var_export``` 等函數打印會寫入到 ```Worker::$stdoutFile``` 指定的檔案中。請注意 ```Worker::$stdoutFile``` 指定的路徑需要有可寫權限。

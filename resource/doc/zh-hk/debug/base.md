# 基本調試

WorkerMan有兩種運行模式，調試模式以及daemon運行模式

運行 ```php start.php start``` 進入調試模式，這時程式碼中的```echo、var_dump、var_export```等函式輸出會在終端機顯示。注意以```php start.php start```運行的WorkerMan在終端機關閉時所有進程會退出。

而運行 ```php start.php start -d``` 則是進入daemon模式，也就是正式上線的運行模式，關閉終端不受影響。

如果想daemon方式運行時也能看到```echo、var_dump、var_export```等函式輸出，可以設置Worker::$stdoutFile屬性，例如

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 將螢幕打印輸出到Worker::$stdoutFile指定的文件中
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```

這樣所有的```echo、var_dump、var_export```等函式輸出會寫入到```Worker::$stdoutFile```指定的文件中。注意```Worker::$stdoutFile```指定的路徑要有可寫權限。

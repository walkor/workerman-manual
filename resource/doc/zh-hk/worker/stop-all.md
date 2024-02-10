# stopAll
```php
void Worker::stopAll(void)
```

停止當前進程並退出。

> **注意**
> `Worker::stopAll()`用於停止當前進程，當前進程退出後主進程會立刻拉起一個新的進程。如果你想停止整個workerman服務，請調用`posix_kill(posix_getppid(), SIGINT)`

### 參數
無參數



### 返回值
無返回

## 范例 max_request

下面例子子進程每處理完1000個請求後執行 stopAll 退出，以便重新啟動一個全新進程。類似php-fpm的max_request屬性，主要用於解決php業務代碼bug引起的內存泄漏問題。

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 每個進程最多執行1000個請求
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // 已經處理請求數
    static $request_count = 0;

    $connection->send('hello http');
    // 如果請求數達到1000
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * 退出當前進程，主進程會立刻重新啟動一個全新進程補充上來
         * 從而完成進程重啟
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```

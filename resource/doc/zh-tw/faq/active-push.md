# 如何主動推送訊息

1、可以使用定時器定時推送資料
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// 進程啟動後定時推送資料給客戶端
$worker->onWorkerStart = function($worker){
    Timer::add(1, function()use($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2、在其他專案中發生某個事件時通知workerman推送資料
參見 [常見問題-在其他專案中推送](push-in-other-project.md)

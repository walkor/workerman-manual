# 메시지를 주도적으로 푸시하는 방법

1. 타이머를 사용하여 데이터를 정기적으로 푸시할 수 있습니다.
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// 프로세스가 시작되면 클라이언트에 정기적으로 데이터를 푸시합니다.
$worker->onWorkerStart = function($worker){
    Timer::add(1, function()use($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2. 다른 프로젝트에서 이벤트가 발생할 때 Workerman에 데이터를 푸시하도록 알리는 것이 가능합니다.
자세한 내용은 [FAQ-다른 프로젝트에서 푸시하는 방법](push-in-other-project.md)를 참조하세요.

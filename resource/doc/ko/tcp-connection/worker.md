# worker
## 설명:
```php
Worker Connection::$worker
```

이 속성은 읽기 전용이며 현재 connection 객체가 속한 worker 인스턴스입니다.


## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// 클라이언트가 데이터를 보낼 때 현재 프로세스에 유지되는 다른 모든 클라이언트에게 데이터를 전달합니다.
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// worker 실행
Worker::runAll();
```

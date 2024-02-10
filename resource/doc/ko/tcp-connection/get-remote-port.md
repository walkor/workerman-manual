# getRemotePort
## 설명:
```php
int Connection::getRemotePort()
```

해당 연결의 클라이언트 포트를 가져옵니다.

## 매개변수

매개변수 없음

## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "주소 " . $connection->getRemoteIp() . ":". $connection->getRemotePort() . "에서 새로운 연결이 이루어졌습니다.\n";
};
// worker 실행
Worker::runAll();
```

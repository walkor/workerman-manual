# getRemoteIp
## 설명:
```php
string Connection::getRemoteIp()
```

해당 연결의 클라이언트 IP 주소를 가져옵니다.

## 파라미터

파라미터 없음

## 예제

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "ip " . $connection->getRemoteIp() . "에서 새로운 연결이 이루어졌습니다.\n";
};
// worker 실행
Worker::runAll();
```

# destroy
## 설명:
```php
void Connection::destroy()
```

연결을 즉시 종료합니다.

close와 다른 점은 destroy를 호출한 후에도 해당 연결의 송신 버퍼에 아직 전송되지 않은 데이터가 있더라도 연결이 즉시 종료되고 연결의 ```onClose```콜백이 즉시 트리거됩니다.

## 매개변수

매개변수 없음

## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // if something wrong
    $connection->destroy();
};
// worker 실행
Worker::runAll();
```

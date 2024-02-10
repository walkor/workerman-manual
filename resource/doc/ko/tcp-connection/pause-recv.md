# pauseRecv
## 설명:
```php
void Connection::pauseRecv(void)
```

현재 연결을 데이터 수신을 중지합니다. 해당 연결의 onMessage 콜백이 트리거되지 않습니다. 이 메서드는 업로드 트래픽 제어에 매우 유용합니다.

## 매개변수

매개변수 없음

## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // connection 객체에 현재 연결이 보낸 요청 수를 저장하는 동적 속성을 추가합니다.
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 각 연결이 100개의 요청을 수신한 후 데이터를 더 이상 수신하지 않도록 합니다.
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
    }
};
// worker 실행
Worker::runAll();
```

## 참고
void Connection::resumeRecv(void) - 해당 연결 객체가 데이터 수신을 다시 시작하도록 합니다.

# resumeRecv
## 설명:
```php
void Connection::resumeRecv(void)
```

현재 연결이 데이터를 계속 수신하도록합니다.이 방법은 Connection::pauseRecv와 함께 사용되며 업로드 트래픽 제어에 매우 유용합니다.

## 매개변수

매개변수 없음

## 예시

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // connection 객체에 현재 연결이 보낸 요청 수를 저장하는 동적 속성 추가
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 각 연결은 100개의 요청을 받은 후 더 이상 데이터를 수신하지 않습니다.
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // 30초 후 다시 데이터를 수신하도록 합니다.
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// worker 실행
Worker::runAll();
```

## 참고
void Connection::pauseRecv(void) - 해당 연결 객체가 데이터 수신을 일시 중지하도록 함

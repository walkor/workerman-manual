# 인증되지 않은 연결 닫기
**질문:**

제한 시간 내에 데이터를 전송하지 않은 클라이언트를 자동으로 닫는 방법은 무엇인가요?
예를 들어 30초 동안 데이터를 수신하지 않으면 해당 클라이언트 연결을 자동으로 닫아서, 인증되지 않은 연결이 규정된 시간 내에 반드시 인증되도록 합니다.

**답변:**

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // 임시로 $connection 객체에 auth_timer_id 속성을 추가하여 타이머 ID 저장
    // 30초 후 연결을 닫기 위해, 클라이언트는 30초 내에 유효성을 확인하고 타이머를 제거해야 합니다.
    $connection->auth_timer_id = Timer::add(30, function() use ($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function(TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
    case 'login':
        ...생략
        // 인증 성공 시, 타이머를 제거하여 연결이 닫히는 것을 방지합니다.
        Timer::del($connection->auth_timer_id);
        break;
         ... 생략
    }
    ... 생략
}
```

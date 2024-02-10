# 연결
## 설명:
```php
array Worker::$connections
```

형식:
```php
array(id=>connection, id=>connection, ...)
```

이 속성은 **현재 프로세스**의 모든 클라이언트 연결 객체를 저장하고 있으며, 여기서 id는 연결의 id 번호이며, 자세한 내용은 [TcpConnection의 id 속성](../tcp-connection/id.md)을 참조하십시오.

`$connections`는 브로드캐스트나 연결 id를 기반으로 연결 객체를 얻을 때 유용합니다.

만약 연결의 번호인 `$id`를 알고 있다면, `$worker->connections[$id]`를 통해 해당 연결 객체를 손쉽게 얻을 수 있으며, 소켓 연결에 대해 작업할 수 있습니다. 예를 들어, `$worker->connections[$id]->send('...')`를 통해 데이터를 보낼 수 있습니다.

주의: 연결이 닫힐 때(onClose가 트리거될 때), 해당 `connection`은 `$connections` 배열에서 삭제됩니다.

주의: 개발자는 이 속성을 수정해서는 안 되며, 그렇게 하면 예상치 못한 상황이 발생할 수 있습니다.

## 예제

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2020');
// 프로세스 시작시 타이머를 설정하여 일정 간격으로 모든 클라이언트 연결에 데이터를 보냅니다.
$worker->onWorkerStart = function($worker)
{
    // 매 10초마다 실행
    Timer::add(10, function()use($worker)
    {
        // 현재 프로세스의 모든 클라이언트 연결을 순회하여 현재 서버의 시간을 보냅니다.
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// worker 실행
Worker::runAll();
```

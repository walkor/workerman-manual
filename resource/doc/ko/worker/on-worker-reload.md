# onWorkerReload
요구조건 ```(workerman >= 3.2.5)```
## 설명:
```php
callback Worker::$onWorkerReload
```
이 기능은 일반적으로 사용되지 않습니다.

Worker가 reload 신호를 받은 후 실행될 콜백을 설정합니다.

onWorkerReload 콜백을 사용하여 비즈니스 구성 파일을 다시로드할 때와 같이 많은 작업을 수행할 수 있습니다.

**주의**:

기본적으로 하위 프로세스는 reload 신호를 받은 후 종료하고 다시 시작하여 새로운 프로세스가 비즈니스 코드를 다시로드하여 코드 업데이트를 완료합니다. 따라서 reload 후 하위 프로세스가 onWorkerReload 콜백을 실행한 후 즉시 종료하는 것은 정상적인 현상입니다.

reload 신호를 받은 후 하위 프로세스가 onWorkerReload만 실행하고 종료하지 않도록 하려면 Worker 인스턴스를 초기화할 때 해당 Worker 인스턴스의 reloadable 속성을 false로 설정할 수 있습니다.

## 콜백 함수의 매개변수

```$worker```

Worker 객체

## 예제

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// reloadable을 false로 설정하여 하위 프로세스가 reload 신호를 받아도 다시 시작하지 않음
$worker->reloadable = false;
// reload 실행 후 모든 클라이언트에게 서버가 reload되었음을 알림
$worker->onWorkerReload = function(Worker $worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// worker 실행
Worker::runAll();
```

참고: 익명 함수를 콜백으로 사용하는 것 외에도 [여기](../faq/callback_methods.md)에서 다른 콜백 작성 방법을 참고할 수 있습니다.

# reloadable
## 설명:
```php
bool Worker::$reloadable
```

`php start.php reload`를 실행하면 모든 하위 프로세스에 리로드 신호(SIGUSR1)가 전송됩니다.

하위 프로세스는 리로드 신호를 받으면 자동으로 종료되고, 그 후에 주 프로세스가 새로운 프로세스를 자동으로 시작합니다. 일반적으로는 비즈니스 코드를 업데이트할 때 사용됩니다.

프로세스의 $reloadable 속성이 false인 경우, 리로드 신호를 받아도 현재 프로세스를 다시 시작시키지 않고, [onWorkerReload](on-worker-reload.md) 이벤트만 발생합니다.

예를 들어, Gateway/Worker 모델에서 gateway 프로세스는 클라이언트 연결을 유지하는 역할을 하고, worker 프로세스는 요청을 처리하는 역할을 합니다. 
gateway 프로세스의 reloadable 속성을 false로 설정하면 클라이언트 연결을 끊지 않고도 업무 코드를 업데이트할 수 있습니다.


## 예제

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 이 인스턴스가 리로드 신호를 받은 후에 다시 시작할지 여부 설정
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// worker 실행
Worker::runAll();
```

# globalEvent

## 설명:
```php
static Event Worker::$globalEvent
```

이 속성은 전역 정적 속성으로, 전역 event loop 인스턴스로 파일 디스크립터의 읽기/쓰기 이벤트 또는 시그널 이벤트를 등록할 수 있습니다.


## 예시

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'Pid is ' . posix_getpid() . "\n";
    // 프로세스가 SIGALRM 시그널을 받았을 때 일부 정보를 출력합니다.
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Get signal SIGALRM\n";
    });
};
// worker 실행
Worker::runAll();
```

## 테스트
Workerman이 시작되면 현재 프로세스 pid(숫자)가 표시됩니다. 명령줄에서 다음을 실행하십시오.
```shell
kill -SIGALRM 프로세스pid
```
서버는 다음과 같이 출력합니다.
```shell
Get signal SIGALRM
```

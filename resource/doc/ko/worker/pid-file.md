# pidFile
## 설명:
```php
static string Worker::$pidFile
```

특별한 필요가 없는 경우에는이 속성을 설정하지 않는 것이 좋습니다.

이 속성은 전역 정적 속성으로, WorkerMan 프로세스의 pid 파일 경로를 설정하는 데 사용됩니다.

이 설정은 감시에 유용하며, WorkerMan의 pid 파일을 고정된 디렉토리에 넣으면 일부 감시 소프트웨어가 pid 파일을 읽어 WorkerMan 프로세스 상태를 모니터링할 수 있습니다.

설정하지 않으면 WorkerMan은 기본적으로 Workerman 디렉토리와 동일한 위치에(pid 파일을 생성하며, 여러 WorkerMan 인스턴스를 시작할 때 pid 충돌을 피하기 위해 WorkerMan은 현재 WorkerMan의 경로도 포함하는 pid 파일을 생성합니다.

참고: 이 속성은 ```Worker::runAll();```가 실행되기 전에 설정해야만 유효합니다. Windows 시스템에서는이 기능을 지원하지 않습니다.


## 예시

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$pidFile = '/var/run/workerman.pid';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// worker 실행
Worker::runAll();
```

# stdoutFile
## 설명:
```php
static string Worker::$stdoutFile
```

이 속성은 전역 정적 속성으로, 데몬 모드(```-d```로 시작)로 실행되는 경우 모든 터미널 출력(echo var_dump 등)은 stdoutFile로 지정된 파일로 리다이렉션됩니다.

만약 설정하지 않고 데몬 모드로 실행되는 경우 모든 터미널 출력은 `/dev/null`로 리다이렉션됩니다 (즉, 모든 출력이 기본적으로 삭제됨).

> 참고: `/dev/null`은 리눅스에서의 특수 파일로, 사실상은 블랙홀입니다. 이 파일로 데이터를 쓰면 모두 삭제됩니다. 출력을 삭제하고 싶지 않은 경우, `Worker::$stdoutFile`을 정상적인 파일 경로로 설정할 수 있습니다.

> 참고: 이 속성은 ```Worker::runAll();```이 실행되기 전에 설정해야만 합니다. Windows 시스템에서는 이 기능을 지원하지 않습니다.

## 예시

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// 모든 출력은 /tmp/stdout.log 파일에 저장됩니다.
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// worker 실행
Worker::runAll();
```

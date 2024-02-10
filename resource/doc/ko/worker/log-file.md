# 로그 파일
## 설명:
```php
static string Worker::$logFile
```

workerman 로그 파일의 위치를 지정하는 데 사용됩니다.

이 파일에는 workerman 자체에 관련된 로그가 기록됩니다. 이는 시작 및 정지 등을 포함합니다.

설정되지 않은 경우 파일 이름은 기본적으로 workerman.log이며 파일 위치는 Workerman의 상위 디렉토리에 있습니다.

**주의:**

이 로그 파일에는 workerman 자체에 관련된 시작 및 정지 등의 로그만 기록되며, 비즈니스 로그는 포함되지 않습니다.

비즈니스 로그는 [file_put_contents](https://php.net/manual/zh/function.file-put-contents.php) 또는 [error_log](https://php.net/manual/zh/function.error-log.php)와 같은 함수를 사용하여 직접 구현할 수 있습니다.

## 예시

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// worker 실행
Worker::runAll();
```

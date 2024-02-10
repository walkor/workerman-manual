# 기본 디버깅

WorkerMan에는 디버그 모드와 데몬 실행 모드 두 가지가 있습니다.

```php start.php start```를 실행하여 디버그 모드에 진입하면 코드 내의 `echo, var_dump, var_export`와 같은 함수 출력이 터미널에 표시됩니다. ```php start.php start```로 실행된 WorkerMan은 터미널을 닫을 때 모든 프로세스가 종료됩니다.

반면에 ```php start.php start -d```를 실행하여 데몬 모드에 진입하면 실제 온라인 모드가 됩니다. 터미널을 닫아도 영향을 받지 않습니다.

만약 데몬 방식으로 실행하면서도 `echo, var_dump, var_export`와 같은 함수 출력을 보고 싶다면 Worker::$stdoutFile 속성을 설정할 수 있습니다. 예를 들면,

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 화면 출력을 Worker::$stdoutFile이 지정한 파일로 보냅니다.
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```

이렇게 하면 모든 `echo, var_dump, var_export`와 같은 함수 출력이 `Worker::$stdoutFile`이 지정한 파일로 기록됩니다. 주의할 점은 `Worker::$stdoutFile`이 지정한 경로에 쓰기 권한이 있어야 합니다.

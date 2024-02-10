# 데몬화
## 설명:
```php
static bool Worker::$daemonize
```

이 속성은 전역 정적 속성으로, 데몬(백그라운드) 모드로 실행할 지 여부를 나타냅니다. 시작 명령에 ```-d``` 매개변수를 사용하면이 속성이 자동으로 true로 설정됩니다. 또한 코드에서 수동으로 설정할 수도 있습니다.

참고: 이 속성은 ```Worker::runAll();```가 실행되기 전에 설정해야 유효합니다. Windows 시스템에서는이 기능을 지원하지 않습니다.

## 예제

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// worker 실행
Worker::runAll();
```

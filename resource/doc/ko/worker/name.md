# 이름

## 설명:
```php
string Worker::$name
```

현재 Worker 인스턴스의 이름을 설정하여 프로세스를 식별하는 데 도움이 됩니다. 설정되지 않은 경우 기본값은 'none'입니다.


## 예제

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 인스턴스 이름 설정
$worker->name = 'MyWebsocketWorker';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// worker 실행
Worker::runAll();
```

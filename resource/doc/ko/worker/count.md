# count

## 설명:
```php
int Worker::$count
```

현재 Worker 인스턴스에서 몇 개의 프로세스를 시작할지 설정합니다. 설정되지 않으면 기본값은 1입니다.

프로세스 수를 설정하는 방법은 [**여기**](../faq/processes-count.md)를 참조하십시오.

참고: 이 속성은 ```Worker::runAll();```가 실행되기 전에 설정해야만 유효합니다. Windows 시스템은이 기능을 지원하지 않습니다.

## 예시

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 8 개의 프로세스 시작, 동시에 8484 포트를 수신 대기하며 websocket 프로토콜을 통해 서비스 제공
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// worker 실행
Worker::runAll();
```
